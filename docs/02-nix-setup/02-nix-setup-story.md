---
title: "Nix Setup Story — Construye tu base declarativa"
version: "1.3"
status: "Versión validada (macOS Tahoe 26.0.1)"
author: "Jyr"
date: "2025-10-18"
description: "Materialización práctica del fundamento declarativo del Reproducible Dev Framework. Instala Nix, integra Direnv y Home Manager, y declara tu primer entorno reproducible."
framework: "Reproducible Dev Framework"
narrative_system: "Story-Driven Diátaxis"
---

<!--  
Contexto interno del sistema narrativo  
Este documento pertenece a la Capa 1 — Fundamento (Nix + Direnv) del Story-Driven Diátaxis  
del proyecto Reproducible Dev Framework.  
-->

# Nix Setup Story — Construye tu base declarativa

> Dejas de limpiar, empiezas a construir.  
> Este capítulo no instala herramientas: declara tu entorno.

---

**Tabla de contenidos**

1. [Introducción](#1-introducción)  
2. [Propósito de esta capa](#2-propósito-de-esta-capa)  
2.1 [Convenciones de workspace (`~/dev/`)](#21-convenciones-de-workspace-dev)  
3. [Instalación de Nix (multiusuario)](#3-instalación-de-nix-multiusuario)  
4. [Integración de Direnv + Flake minimal](#4-integración-de-direnv--flake-minimal)  
5. [Declarar Home Manager (nivel usuario)](#5-declarar-home-manager-nivel-usuario)  
6. [Primer devShell funcional](#6-primer-devshell-funcional)  
7. [Validaciones y señales de éxito](#7-validaciones-y-señales-de-éxito)  
8. [Cierre narrativo](#8-cierre-narrativo)

---

## 1. Introducción

**Propósito técnico:** iniciar la fase práctica del Reproducible Dev Framework.  
**Reflexión:** lo que antes fue limpieza, ahora se transforma en construcción consciente.  
Instalar Nix no es agregar una herramienta; es declarar tu relación con el entorno.  

A partir de aquí, cada decisión técnica debe tener un propósito y un registro.  
Tu entorno dejará de depender de comandos aislados y comenzará a existir como una declaración reproducible.

---

## 2. Propósito de esta capa

**Propósito técnico:** materializar los principios definidos en Foundations.  
**Reflexión:** esta capa es el punto de no retorno.  
Si Reset Story borró lo accidental y Foundations dio sentido, Nix Setup Story crea el suelo sobre el cual todo se sostendrá.

En esta etapa aprenderás a:
- Instalar Nix de forma multiusuario (control total).  
- Activar Direnv y Flake para aislar entornos.  
- Declarar Home Manager para tu configuración personal.  
- Crear tu primer `devShell` reproducible.  

No busques instalar todo; busca entender por qué instalas cada cosa.

---

## 2.1 Convenciones de workspace (`~/dev/`)

**Propósito:** estandarizar dónde vive tu trabajo reproducible.

### Estructura recomendada
```bash
mkdir -p ~/dev/{lab,personal,work,hm}
```

- `~/dev/lab/` → **experimentos desechables**: spikes, PoC, demos (p. ej. `asdf-demo`, `just-demo`, `first-shell`).  
- `~/dev/personal/` → **proyectos personales** estables (p. ej. `dotfiles`, `cli-tools`).  
- `~/dev/work/` → **proyectos laborales** (repos corporativos).  
- `~/dev/hm/` → **flake de Home Manager** (capa de usuario; no es un proyecto).  

> Beneficios: rutas predecibles, onboarding más fácil, cero colisiones de estados y separación clara entre **capa usuario (HM)** y **capa proyecto (flakes por repo)**.

### Convenciones mínimas
- **Un repo = una carpeta** (sin mezclar múltiples repos dentro de una sola carpeta).  
- Cada repo **contiene su `flake.nix` + `.envrc`** (y opcionalmente `Justfile`) → *autoactivación* con `direnv`.  
- **Nada** en `~/.asdf` ni `~/.tool-versions` globales: si usas ASDF, define `ASDF_DATA_DIR="$PWD/.asdf"` en **cada proyecto**.  
- Si el repo usa `just`, el `Justfile` **se versiona** junto al código (no generado desde Nix).  

### Ejemplos de rutas (usa estas bases en todo el documento)
- Demos/guías: `~/dev/lab/<demo-name>`  
- Proyectos propios estables: `~/dev/personal/<project>`  
- Proyectos de trabajo: `~/dev/work/<project>`  
- Flake de Home Manager (usuario): `~/dev/hm`  

> Cuando veas rutas de ejemplo, aplícalas bajo `~/dev/` según la intención del repo.

---

## 3. Instalación de Nix (multiusuario)

### Propósito técnico  
Establecer una instalación limpia y controlada de Nix en macOS (Apple Silicon).

### Reflexión  
Nix no se instala: se declara como gestor de decisiones.  
Cada comando que ejecutas aquí es el inicio de una práctica declarativa: saber exactamente qué versión tienes, dónde vive y cómo se reproduce.

---

### 3.1 Comando reproducible

```bash
sh <(curl -L https://nixos.org/nix/install) --daemon
```

**Notas:**
- Usa siempre la opción `--daemon` (modo multiusuario).  
- Si aparece un error de respaldo de `/etc/bashrc.backup-before-nix` o `/etc/zshrc.backup-before-nix`, muévelo:
  ```bash
  sudo mv /etc/zshrc.backup-before-nix /etc/zshrc.backup-before-nix.bak
  sudo mv /etc/bashrc.backup-before-nix /etc/bashrc.backup-before-nix.bak
  ```
- Recarga la sesión:
  ```bash
  exec $SHELL -l
  ```

---

### 3.2 Verificar instalación inicial

```bash
nix --version
nix config check
```

> En una instalación limpia de macOS es normal obtener este mensaje:
> ```
> error: experimental Nix feature 'nix-command' is disabled; add '--extra-experimental-features nix-command' to enable it
> ```
> Esto ocurre porque Nix no habilita por defecto las funciones `nix-command` y `flakes`, que permiten ejecutar comandos declarativos y gestionar flakes.

---

### 3.3 Habilitar las features experimentales

Puedes hacerlo de dos formas:

**Solución inmediata (solo esta sesión):**
```bash
nix --extra-experimental-features 'nix-command flakes' config check
```

**Solución permanente (recomendada):**
```bash
sudo mkdir -p /etc/nix
echo "experimental-features = nix-command flakes fetch-tree" | sudo tee /etc/nix/nix.conf
sudo launchctl kickstart -k system/org.nixos.nix-daemon
exec $SHELL -l
```

Valida que quedó configurado:
```bash
nix config show | sed -n 's/^experimental-features = //p'
```

Debes ver al menos:
```
nix-command flakes
```

> Es posible que también aparezca `fetch-tree`, lo cual es normal y seguro:  
> Nix lo habilita en versiones recientes para mejorar el manejo de flakes.

---

**Reflexión:**  
Este error no es un fallo, es una invitación a tomar control.  
Habilitar estas funciones marca el momento en que Nix deja de ser “una herramienta”  
y se convierte en tu sistema de gestión declarativa.

---

## 4. Integración de Direnv + Flake minimal

> **Propósito narrativo:**  
> Aquí ocurre el primer momento de autonomía real del sistema.  
> Dejas de ejecutar comandos manuales y permites que el entorno se configure solo.  
> Es el instante donde el “cd” deja de ser un cambio de carpeta y se convierte en un acto declarativo.

---

### 4.1 Comprensión

Direnv es la capa que convierte el **movimiento** en **contexto**.  
Cada directorio es un universo con sus propias reglas.  
Entrar a un proyecto no solo cambia de ruta: **activa un entorno reproducible**  
que Nix prepara en silencio cada vez que lo visitas.

> Entra, trabaja, sal, y el entorno se apaga.  
> No deja rastros. No rompe nada.

---

### 4.2 Crear tu primer entorno declarativo

Asegúrate de estar en tu laboratorio y crea el proyecto de práctica:

```bash
mkdir -p ~/dev/lab/direnv-demo
cd ~/dev/lab/direnv-demo
```

Crea el archivo `.envrc` (Direnv lo leerá para activar el flake **cuando exista**):

```bash
printf "use flake\n" > .envrc
```

> **Aún no autorices** con `direnv allow`; primero define el flake  
> para que la autorización tenga un propósito claro.

---

### 4.3 Definir el flake minimal

Crea `flake.nix` con la mínima expresión de un entorno que se autoconfigura:

```nix
{
  description = "Primer entorno reproducible con direnv y Nix (minimal)";

  inputs.nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";

  outputs = { nixpkgs, ... }: {
    devShells.aarch64-darwin.default = let
      pkgs = import nixpkgs { system = "aarch64-darwin"; };
    in
      pkgs.mkShell {
        buildInputs = [ pkgs.hello ];
        shellHook = ''
          echo "✅ Entorno reproducible activo"
        '';
      };
  };
}
```

> Este flake es intencionalmente simple.  
> Es la mínima señal de que **el sistema responde al contexto**.

---
**Validar comportamiento**

Comprueba que el binario `hello` proviene del Nix Store y responde:

```bash
which hello
hello
```

🖥️ **Resultado esperado:**
```
/nix/store/...-hello-<versión>/bin/hello
Hello, world!
```

---

### 4.3.1 Evolución del flake: incluir `direnv` sin instalaciones globales

Sin romper la filosofía (“Nada global”), extiende el `devShell` para que **`direnv` esté disponible dentro del entorno**.  
Modifica el bloque `buildInputs` de tu `flake.nix` a:

```nix
buildInputs = with pkgs; [
  direnv    # disponible DENTRO del devShell (sin instalación global)
  hello
];
```

> Con esto podrás **autorizar** el `.envrc` usando `direnv` desde el propio `devShell`,  
> sin depender todavía de Home Manager. La auto-activación total llegará más adelante.

---

### 4.4 Autorizar y activar

Ahora sí, autoriza **este directorio**.  
Si aún no tienes `direnv` instalado a nivel usuario, invócalo **desde el devShell**:

```bash
nix develop -c direnv allow
```

Verifica el estado:

```bash
nix develop -c direnv status
```

🖥️ **Salida esperada:**
```
Found RC allowed 0
```

> En esta etapa, el valor `0` indica que `.envrc` fue reconocido,  
> pero aún **no existe una integración activa** porque `direnv` no está configurado globalmente.  
> Esto es correcto y esperado: la activación automática llegará más adelante  
> cuando declares `direnv` con **Home Manager** (capa usuario).  
>
> Mientras tanto, puedes entrar al entorno manualmente con:
>
> ```bash
> nix develop
> ```
>
> Verás: `✅ Entorno reproducible activo`.

---

### 4.5 Reflexión narrativa

> Este paso no trata de “usar direnv”, sino de **entender cómo declarar el movimiento**.  
>  
> Has pasado de ejecutar a orquestar.  
> Tu sistema ahora es consciente del espacio en el que estás,  
> y cada directorio puede tener su propio propósito, sin invadir a los demás.  
>  
> Aquí nace el verdadero flujo reproducible:  
> cada proyecto es una declaración viva, y cada flake, un acto de intención.

---

### 4.6 Validaciones rápidas

| Comando                            | Propósito                         | Resultado esperado                                  |
|------------------------------------|-----------------------------------|-----------------------------------------------------|
| `nix develop -c direnv status`     | Estado del hook                   | `.envrc` reconocido (`Found RC allowed 0`)          |
| `nix develop`                      | Ingreso manual al devShell        | Muestra “✅ Entorno reproducible activo”            |
| `which hello`                      | Verifica el binario               | Ruta en `/nix/store/.../bin/hello`                  |

---

✅ **Resultado final:**  
Primer entorno reproducible activo, **sin instalaciones globales**.  
Todo vive dentro del proyecto, bajo `~/dev/lab/direnv-demo`.

> Has cruzado el umbral entre el control manual y la automatización declarativa.  
> En el siguiente capítulo declararemos `direnv` con Home Manager para habilitar la **auto-activación** al entrar en cada repo.

---

##  5. Declarar Home Manager (nivel usuario)

###  Propósito narrativo

Has estado trabajando dentro de tu laboratorio personal (`~/dev/lab`), donde cada proyecto vive y respira de forma independiente.  
Pero en este punto, algo empieza a hacerse evidente: no solo necesitas proyectos reproducibles, también necesitas **un entorno personal reproducible**.  

> Aquí entra Home Manager: tu capa de identidad como desarrollador.  
> No administra tus proyectos, sino **tu entorno base**.  
> Es el lugar donde declaras qué herramientas acompañan tu día a día,  
> y donde cada sesión se comporta de la misma manera sin importar el contexto.

---

###  Objetivo técnico

Declarar **Home Manager** como la capa que controla tus configuraciones personales  
(`direnv`, `git`, `neovim`, etc.), garantizando que:

- Todo esté **bajo control declarativo**, dentro de un `flake.nix`.
- No haya configuraciones manuales o globales dispersas.
- Tu entorno de usuario pueda **reconstruirse en segundos** en cualquier Mac.

---

###  Estructura de directorios antes de comenzar

Hasta ahora, tu estructura en `~/dev/` debería lucir así:

```
~/dev/
├── hm/          ← aquí vivirá tu configuración declarativa de usuario
└── lab/         ← aquí viven tus proyectos experimentales y personales
```

> 📍 **Importante:**  
> Home Manager se configura en `~/dev/hm`.  
> Ese directorio no pertenece a ningún proyecto;  
> es el espacio donde describes quién eres como desarrollador dentro de Nix.

---

### 5.1 Crear el directorio de Home Manager

Entra en tu directorio base de desarrollo y crea la carpeta para Home Manager:

```bash
mkdir -p ~/dev/hm
cd ~/dev/hm
```

💡 **Por qué aquí:**  
Este paso define un patrón:  
todo lo que sea de tu entorno personal vive en `hm`,  
todo lo que sea experimental o de trabajo vive en `lab`.

> “Declarar tu entorno es el primer paso para dejar de configurarlo manualmente.”

---

###  5.2 Crear el flake de Home Manager

Ahora escribirás tu primer **flake declarativo de usuario**.  
Este archivo le dice a Nix quién eres, dónde vives en tu sistema,  
y qué herramientas deben estar disponibles cada vez que abres una terminal.

Crea `flake.nix` dentro de `~/dev/hm` con este contenido:

```nix
{
  description = "Declarative per-user setup (macOS, Apple Silicon) with Home Manager";

  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
    home-manager.url = "github:nix-community/home-manager";
    home-manager.inputs.nixpkgs.follows = "nixpkgs";
  };

  outputs = { self, nixpkgs, home-manager, ... }:
  let
    system = "aarch64-darwin";
    pkgs = import nixpkgs { inherit system; };
  in {
    homeConfigurations.YOUR_USER = home-manager.lib.homeManagerConfiguration {
      inherit pkgs;
      modules = [
        {
          home.username = "YOUR_USER";
          home.homeDirectory = "/Users/YOUR_USER";

          programs.zsh.enable = true;

          programs.direnv = {
            enable = true;            # instala direnv de forma declarativa
            nix-direnv.enable = true; # integra flakes y cachea evaluaciones
          };

          home.stateVersion = "24.05";
        }
      ];
    };
  };
}
```

> 📘 **Nota técnica:**  
> `home.stateVersion` no cambia tu sistema operativo,  
> solo indica qué versión de Home Manager estás usando como base.

---

###  5.3 Aplicar la configuración declarativa

Ejecuta tu primer **Home Manager switch**:  

```bash
nix run home-manager/master -- switch --flake ~/dev/hm#YOUR_USER
```

🧠 Esto hace tres cosas:
1. Instala Home Manager si aún no existía.  
2. Aplica tu configuración declarativa (flake).  
3. Activa automáticamente el *hook* de `direnv` en `zsh`.

> “De ahora en adelante, no editas tu `.zshrc`: lo declaras.”

---

###  5.4 Recargar la sesión de shell

```bash
exec $SHELL -l
```

Esto vuelve a cargar tu sesión con el hook de `direnv` activo.  
No necesitas modificar rutas ni variables manualmente:  
Home Manager ya lo hizo por ti.

---

###  5.5 Validar instalación de `direnv`

```bash
which direnv
direnv version
```

🖥️ **Salida esperada:**
```
/nix/store/.../bin/direnv
direnv v2.x.x
```

Si ves algo diferente (por ejemplo, `command not found`),  
repite el comando del paso anterior y asegúrate de recargar la sesión.

---

###  5.6 Validar activación automática del entorno reproducible

Dirígete a tu primer proyecto (`direnv-demo`)  
y autoriza su `.envrc`:

```bash
cd ~/dev/lab/direnv-demo
direnv allow
direnv status
```

🖥️ **Salida esperada:**
```
Found RC allowed true
```

> Si aparece `Found RC allowed 0`, no te preocupes:  
> significa que el `.envrc` aún no ha sido autorizado;  
> vuelve a ejecutar `direnv allow` dentro del directorio del proyecto.

Cada vez que entres en `~/dev/lab/direnv-demo`,  
verás cómo el entorno se activa automáticamente gracias al hook de Home Manager.

---

###  Cierre narrativo

Con este paso terminaste algo fundamental:  
**Home Manager ya controla tu entorno personal.**

Tu usuario dejó de depender de configuraciones ocultas o globales.  
Ahora cada herramienta, variable o hook se describe, versiona y puede reproducirse.

> “El caos de la configuración manual se convierte en un manifiesto reproducible.”

---

## 6. Primer entorno funcional: Home Manager + devShell

###  Propósito narrativo
Hasta aquí ya definiste tu filosofía y tus herramientas base.  
Este paso marca la transición de la teoría a la práctica:  
construirás tu **capa personal declarativa (Home Manager)** y tu **primer entorno de proyecto reproducible (devShell)**.

> “Home Manager representa quién eres.  
> El devShell representa lo que estás creando.”

---

## 6.1 Home Manager: tu capa personal declarativa

### Propósito narrativo  
Tu entorno reproducible no solo existe en los proyectos: también en ti.  
Home Manager es tu **capa personal**, la parte del sistema que define cómo trabajas, no qué construyes.

> “Home Manager representa quién eres.  
> Cada herramienta que declaras aquí, es una decisión sobre tu forma de crear.”

En esta etapa comienzas a **dar forma a tu identidad técnica**.  
Cada línea que agregas a Home Manager es una afirmación:  
> “Quiero que mi entorno se configure solo, sin comandos manuales, sin olvidos.”

Este paso no trata solo de instalar programas, sino de **diseñar la experiencia que quieres tener cada vez que abres tu terminal**.

---

### Propósito técnico  
Home Manager controla todo lo que debe estar disponible **siempre que inicias sesión**:  
editores, terminales, versionadores y helpers.  
No se instalan globalmente: se **declaran**, se **aplican**, y viven dentro del **/nix/store**, bajo control total del sistema reproducible.

Desde aquí comenzaremos a instalar herramienta por herramienta,  
empezando por la más importante: **direnv**, la encargada de abrir las puertas de cada proyecto.

---

### Estructura base del entorno
Hasta este punto ya deberías tener la siguiente estructura:

```
~/dev/
└── hm/             ← Configuración declarativa personal
    └── flake.nix   ← Declaraciones de Home Manager
└── lab/            ← Espacio de proyectos experimentales
```

El directorio `hm` es tu **entorno personal**,  
mientras que `lab` será donde vivirán tus **proyectos por flake**.

> Mantener esta separación te permite reconstruir todo tu espacio de trabajo  
> con solo dos flakes: uno personal (Home Manager) y otro por proyecto.

---

### Archivo completo: `~/dev/hm/flake.nix`

> Reemplaza `YOUR_USER` por tu usuario real (`echo $USER`).  
> Este archivo es **auto-contenible**: puedes copiarlo y aplicarlo sin depender de configuraciones previas.

```nix
{
  description = "User layer (Home Manager) — macOS Apple Silicon (aarch64-darwin)";

  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
    home-manager.url = "github:nix-community/home-manager";
    home-manager.inputs.nixpkgs.follows = "nixpkgs";
  };

  outputs = { nixpkgs, home-manager, ... }:
  let
    system = "aarch64-darwin";
    pkgs = import nixpkgs { inherit system; };
  in {
    # Cambia YOUR_USER por tu usuario real
    homeConfigurations.YOUR_USER = home-manager.lib.homeManagerConfiguration {
      inherit pkgs;

      modules = [
        ({ config, pkgs, ... }: {
          home.username = "YOUR_USER";
          home.homeDirectory = "/Users/YOUR_USER";
          home.stateVersion = "24.05";

          # 🧩 Shell base (zsh) — el hook de direnv se conecta aquí
          programs.zsh = {
            enable = true;
            # Refuerza PATH para priorizar binarios de Nix
            initContent = ''
              export PATH="$HOME/.nix-profile/bin:/nix/var/nix/profiles/default/bin:$PATH"
            '';
          };

          # 🧩 Direnv: activa entornos declarados por proyecto (.envrc)
          programs.direnv = {
            enable = true;            # instala direnv declarativamente
            nix-direnv.enable = true; # integra flakes y cachea evaluaciones
          };

          # Puedes añadir más programas en pasos posteriores (git, nvim, etc.)
          # Mantén este paso enfocado únicamente en direnv.
        })
      ];
    };
  };
}
```

---

### Aplicar los cambios y recargar sesión

```bash
nix run home-manager/master -- switch --flake ~/dev/hm#YOUR_USER
exec $SHELL -l
```

🔹 El primer comando aplica toda tu configuración declarada.  
🔹 El segundo recarga tu sesión para que el hook de `direnv` y el PATH queden activos.

> A partir de ahora, cada vez que cambies tu `flake.nix`,  
> bastará con repetir esos dos comandos para actualizar tu entorno.

---

### Validaciones rápidas

Comprueba que `direnv` está correctamente instalado y en el PATH de Nix:

```bash
which direnv
direnv version
type -a direnv
```

🖥️ **Salida esperada:**
```
/nix/store/.../bin/direnv
direnv v2.x.x
direnv is /nix/store/.../bin/direnv
```

> Si la ruta apunta al `/nix/store/`, todo está bien: significa que `direnv` vive dentro del entorno declarativo, no en el sistema global.

---

### Prueba mínima del hook (sin flakes aún)

Ahora vamos a comprobar que `direnv` puede detectar un archivo `.envrc`  
y modificar el entorno de forma segura.

```bash
mkdir -p ~/dev/lab/direnv-demo
cd ~/dev/lab/direnv-demo
printf 'export DIRENV_TEST="ok"\n' > .envrc

direnv status
direnv allow
direnv status
echo "$DIRENV_TEST"
```

🧩 **Señales de éxito:**
- `direnv status` detecta el archivo `.envrc`.  
- Tras ejecutar `direnv allow`, verás:
  ```
  Found RC allowed 0
  ```
  (significa que el archivo `.envrc` fue autorizado y monitoreado).
- `echo "$DIRENV_TEST"` imprime `ok`.

---

### Errores comunes y solución

| Síntoma | Causa probable | Solución |
|---|---|---|
| `direnv: command not found` | Home Manager no aplicado o sesión vieja | Ejecuta el switch y luego `exec $SHELL -l`. |
| `.envrc is blocked` | Falta autorización local | Ejecuta `direnv allow` dentro del directorio. |
| Hook no se dispara al entrar/salir | La shell no recargó el hook | Abre una **nueva** sesión de terminal o ejecuta `exec $SHELL -l`. |
| `Found RC allowed` no aparece | `.envrc` vacío o mal ubicado | Crea `.envrc` en la **raíz** del directorio y repite `direnv allow`. |

---

### Cierre narrativo

Has dado tu primer paso dentro del **entorno personal reproducible**.  
Ahora tu terminal **ya no es solo una ventana**, es un espacio que responde a tus decisiones.  

> “Cada herramienta declarada aquí forma parte de ti.  
> Es la forma en que el sistema te recuerda quién eres cuando programas.”

El siguiente paso será **añadir ASDF declarativamente**:  
la herramienta que hará que cada proyecto tenga su propio ecosistema de versiones,  
sin afectar tu entorno global.

---

## 6.2 ASDF — Binario declarativo y versiones por proyecto

### Propósito narrativo  
Has abierto la puerta con Direnv; ahora necesitas un guardián del tiempo.  
ASDF representa la memoria técnica de tus proyectos:  
qué versión de cada lenguaje usas y en qué contexto.  

> “No recuerda por ti.  
> Te obliga a declarar cada decisión que tomas.”  

En el mundo declarativo, **ASDF es disciplina**:  
no improvisas, escribes las versiones que definen tu trabajo,  
y cada repo se vuelve un entorno cerrado, autosuficiente y predecible.

---

### Propósito técnico  
ASDF se instala de forma **declarativa con Home Manager**,  
pero cada proyecto mantiene sus propias versiones dentro de su carpeta (`.tool-versions` y `.asdf/`).  
Esto asegura un binario común —controlado por Nix— y datos locales —controlados por el repositorio—.

---

### Pasos  

1️⃣ Posiciónate en tu directorio de configuración de Home Manager (`~/dev/hm`):

```bash
cd ~/dev/hm
```

2️⃣ Edita tu archivo `flake.nix` y agrega la siguiente sección dentro del bloque `modules = [ { … } ];`  

> 💡 Home Manager no cuenta con un módulo nativo de ASDF,  
> pero puedes declararlo de manera estable y portable usando `pkgs.asdf-vm`.

```nix
# Declaración estable de ASDF en Home Manager
home.packages = [ pkgs.asdf-vm ];

programs.zsh = {
  enable = true;
  initContent = ''
    # Inicializa asdf-vm de forma declarativa (sin instalación global)
    . ${pkgs.asdf-vm}/share/asdf-vm/asdf.sh

    # Opcional: autocompletado de ASDF
    fpath+=(${pkgs.asdf-vm}/share/zsh/site-functions)
  '';
};
```

3️⃣ Aplica los cambios y recarga tu sesión:

```bash
nix run home-manager/master -- switch --flake ~/dev/hm#YOUR_USER
exec $SHELL -l
```

---

### Validaciones

Comprueba que ASDF proviene de **Nix** (no de Homebrew ni de una instalación manual):

```bash
which asdf
asdf --version
type -a asdf
```

🖥️ **Salida esperada (ejemplo real):**
```text
asdf () {
  case $1 in
  ("shell") if ! shift
    then
      printf '%s\n' 'asdf: Error: Failed to shift' >&2
      return 1
    fi
    eval "$(asdf export-shell-version sh "$@")" ;;
  (*) command asdf "$@" ;;
  esac
}
v0.15.0
asdf is a shell function from /nix/store/...-asdf-vm-0.15.0/share/asdf-vm/asdf.sh
asdf is /nix/store/...-asdf-vm-0.15.0/share/asdf-vm/bin/asdf
```

🔍 **Cómo interpretar la salida:**

- `asdf is a shell function ...`  
  → Significa que Zsh cargó ASDF desde la ruta declarada en `initContent`.  
  Es el comportamiento correcto cuando ASDF se declara con Home Manager.

- `asdf is /nix/store/.../asdf-vm-0.15.0/.../bin/asdf`  
  → Confirma que el binario proviene del **Nix Store**, no de `/opt/homebrew` ni `/usr/local`.  
  Si aparece una ruta distinta, elimina la instalación anterior antes de continuar.

- `v0.15.0`  
  → Verifica que ASDF se está ejecutando correctamente desde Nix.

💡 Si ves rutas de Homebrew (`/opt/homebrew/...`) o manuales (`/usr/local/...`), ejecuta:

```bash
brew uninstall asdf || true
hash -r
exec $SHELL -l
```

y vuelve a aplicar Home Manager:

```bash
nix run home-manager/master -- switch --flake ~/dev/hm#YOUR_USER
```

---

### Reflexión narrativa  

Tu entorno ya no **instala** herramientas; **declara intenciones**.  
ASDF ahora vive en tu capa personal declarativa — pero no impone versiones globales.  
Cada proyecto podrá declarar sus propias versiones, aisladas, limpias y reproducibles.  

> Direnv le da vida al entorno.  
> ASDF le da memoria.

---

## 6.3 Git — Identidad y control declarativo

### Propósito narrativo  
Has llegado al punto donde tu entorno empieza a tener una identidad propia.  
Git no es solo una herramienta para hacer `commit` y `push`:  
es la memoria de tu trabajo y la forma en que tus decisiones viven fuera de tu máquina.

> “Cada commit es una huella.  
> Si tu entorno es declarativo, tus huellas también deben serlo.”

Aquí defines **quién eres como desarrollador dentro de este sistema reproducible**:
- tu nombre,
- tu correo,
- tu rama inicial,
- tu estilo de diff,
- tus atajos mentales (`lg`, `undo`, `st`…).

La idea es simple:  
lo declaras una vez en Home Manager, y ese “tú” se puede reproducir en cualquier otra Mac sin volver a configurar Git a mano.

Esto deja de ser “mi Mac está configurada así”  
y se convierte en “yo trabajo así, donde sea que esté”.

### Propósito técnico  
Vamos a:
1. Instalar Git usando Nix (no el Git que viene con macOS).
2. Declarar tu configuración global de Git dentro del `flake.nix` de Home Manager.
3. Declarar un archivo global de ignorados (`~/.config/git/ignore`) también desde Nix.
4. Habilitar `delta` como visualizador de diffs moderno.
5. Asegurar que tu PATH use el Git de Nix primero, no el del sistema.

Resultado:  
- Git queda bajo control declarativo.  
- Tu identidad (`user.name`, `user.email`) vive en código.  
- Tu estilo de trabajo se versiona.

### Pasos técnicos

#### 1. Editar tu flake de Home Manager (`~/dev/hm/flake.nix`)

Abre el archivo `~/dev/hm/flake.nix`, localiza el módulo `{ config, pkgs, ... }: { ... }` que define tu entorno de usuario, y asegúrate de tener (o añade) este bloque completo:

```nix
{ config, pkgs, ... }: {
  # 1. Asegura precedencia de PATH
  # Queremos que Git use la versión declarada en Nix,
  # NO el Git que viene con macOS en /usr/bin/git.
  home.sessionPath = [
    "${config.home.homeDirectory}/.nix-profile/bin"
    "/nix/var/nix/profiles/default/bin"
  ];

  programs.zsh = {
    enable = true;
    initContent = ''
      # Refuerza PATH al inicio de cada sesión interactiva.
      export PATH="$HOME/.nix-profile/bin:/nix/var/nix/profiles/default/bin:$PATH"
    '';
  };

  # 2. Instalar Git y herramientas relacionadas
  # Estas herramientas viven en tu capa usuario (Home Manager),
  # no en cada proyecto individual.
  home.packages = with pkgs; [
    git
    git-lfs   # opcional: Large File Storage
  ];

  # 3. Configuración declarativa de Git con la API nueva de Home Manager
  programs.git = {
    enable = true;

    # A partir de las versiones recientes de Home Manager,
    # TODA la configuración va en `settings`.
    settings = {
      # Identidad
      user.name = "Tu Nombre";
      user.email = "tu.email@dominio.tld";

      # Rama inicial y flujo seguro
      init.defaultBranch = "main";
      pull.ff = "only";        # solo fast-forward, evita merges automáticos raros
      push.default = "simple"; # push directo a la rama asociada
      fetch.prune = true;      # limpia refs remotas obsoletas

      # Preferencias de entorno
      core.autocrlf = "input"; # seguro en macOS/Linux
      core.editor = "nvim";    # usa Neovim como editor para mensajes de commit
      core.excludesFile = "${config.xdg.configHome}/git/ignore";

      color.ui = "auto";

      # Usa el llavero del sistema para credenciales HTTPS
      credential.helper = "osxkeychain";

      # Aliases personales
      alias.co   = "checkout";
      alias.br   = "branch";
      alias.st   = "status -sb";
      alias.ci   = "commit";
      alias.ca   = "commit --amend";
      alias.lg   = "log --graph --decorate --oneline --all";
      alias.last = "log -1 HEAD";
      alias.undo = "reset --soft HEAD~1";
    };
  };

  # 4. Delta: diffs modernos, legibles y navegables
  # Home Manager ahora lo maneja como módulo separado (`programs.delta`)
  programs.delta = {
    enable = true;
    enableGitIntegration = true;
    options = {
      navigate = true;
      line-numbers = true;
      side-by-side = true;
      syntax-theme = "Monokai Extended";
    };
  };

  # 5. Archivo global de ignorados (también gestionado por Nix)
  # Esto reemplaza el típico ~/.gitignore_global manual.
  home.file."${config.xdg.configHome}/git/ignore".text = ''
    # Archivos de sistema
    .DS_Store

    # Entornos locales
    .env
    .envrc

    # Node / JS
    node_modules/
    dist/

    # Elixir / Mix
    _build/
    deps/

    # Editores / OS
    .idea/
    .vscode/
    .fleet/
  '';
}
```

Qué estás logrando con esto:
- El binario de `git` viene de Nix.
- El PATH prioriza ese binario.
- Tu identidad (`user.name`, `user.email`) está bajo `settings`, no en tu `~/.gitconfig` manual.
- Tienes alias declarados (no improvisados).
- `delta` transforma el diff para hacerlo legible.
- Tu `~/.config/git/ignore` es generado por Home Manager, no por edición manual.

Este bloque reemplaza por completo la configuración de Git que teníamos antes.  
Ya no usamos:
- `programs.git.userName`
- `programs.git.userEmail`
- `programs.git.extraConfig`
- `programs.git.aliases`
- `programs.git.delta`
Todas esas opciones fueron renombradas en Home Manager y ahora viven bajo `settings` o en `programs.delta`.

#### 2. Aplicar configuración declarativa

Ejecuta:

```bash
nix run home-manager/master -- switch --flake ~/dev/hm#$(whoami)
exec $SHELL -l
```

Qué hace esto:
- Reconstruye tu entorno declarativo de usuario.
- Escribe (o actualiza) `~/.config/git/config` basado en `settings`.
- Crea/actualiza `~/.config/git/ignore`.
- Asegura que el Git de Nix sea visible en tu nueva sesión shell.

Si abriste una nueva ventana de WezTerm justo después, mejor todavía.

#### 3. Validaciones rápidas

Ahora vamos a comprobar tres cosas:
1. Que estás usando el Git que viene de Nix.
2. Que la identidad está aplicada.
3. Que el archivo global de ignorados existe.

Ejecuta:

```bash
which -a git
git --version
git config --global --get user.name
git config --global --get user.email
git config --global --get init.defaultBranch
git config --global --get core.excludesFile
```

Salida esperada (ejemplo):

```text
/Users/tu_usuario/.nix-profile/bin/git
/usr/bin/git
git version 2.51.0
Tu Nombre
tu.email@dominio.tld
main
/Users/tu_usuario/.config/git/ignore
```

Interpretación:
- Si ves primero `/Users/tu_usuario/.nix-profile/bin/git`, perfecto.
- Si ves primero `/usr/bin/git`, significa que sigue tomando el Git que viene con macOS → abre una nueva sesión de terminal para que los cambios del PATH tomen efecto.

---

#### 4. Confirmar archivo global de ignorados

Queremos confirmar dos cosas:
- Que el archivo fue creado por Home Manager.
- Que Git sabe que debe usarlo.

Ejecuta:

```bash
test -f ~/.config/git/ignore && echo "OK: ignore presente" || echo "FALTA ignore"
head -n 10 ~/.config/git/ignore
```

Salida esperada (ejemplo):

```text
OK: ignore presente
# Archivos de sistema
.DS_Store
.env
.envrc
node_modules/
dist/
...
```

Esto demuestra:
- Que Home Manager generó `~/.config/git/ignore`.
- Que `core.excludesFile` lo está apuntando.

Nada de esto se hizo editando `~/.gitconfig` a mano: todo viene del `flake.nix`.

---

#### 5. Validar delta y diffs modernos

`delta` es el motor visual de diffs que hace que `git diff` sea legible.

Validemos que está integrado correctamente:

```bash
git config --global --get delta.side-by-side
git config --global --get delta.line-numbers
git config --global --get delta.navigate
```

Salida esperada (ejemplo):

```text
true
true
true
```

Esto confirma que `programs.delta` se aplicó y que Git ya sabe usar `delta`.

Ahora haz un diff cualquiera dentro de un repo con cambios sin commitear:

```bash
git diff
```

Deberías ver:
- colores,
- números de línea,
- columnas lado a lado (`side-by-side`),
- un layout que ya no parece el diff crudo estándar.

### Problemas comunes

| Síntoma                                                   | Causa probable                                                                 | Solución                                                                                       |
|----------------------------------------------------------|-------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------|
| `git --version` muestra “Apple Git-154”                  | Tu shell sigue usando `/usr/bin/git` en vez de `~/.nix-profile/bin/git`      | Cierra y vuelve a abrir WezTerm, o asegúrate de que `home.sessionPath` y `programs.zsh.initContent` estén en tu flake. |
| `fatal: bad config line … ~/.config/git/ignore`          | Se intentó usar `programs.git.includes` o mezclaste config vieja             | Elimina cualquier uso de `programs.git.includes`. Debes usar solo `core.excludesFile` en `settings`. |
| `git lfs` no existe                                      | No agregaste `git-lfs` a `home.packages`                                     | Añade `git-lfs` y vuelve a aplicar Home Manager.                                              |
| `delta` sin efecto en `git diff`                         | No declaraste `programs.delta.enableGitIntegration = true;`                  | Asegúrate de tener el bloque `programs.delta` exactamente como está arriba y vuelve a aplicar. |
| El primer `which -a git` sigue apuntando a `/usr/bin/git`| La sesión actual sigue con PATH viejo                                        | Abre una nueva sesión de WezTerm (o `exec $SHELL -l`) y vuelve a ejecutar `which -a git`.     |

### Resultado esperado

Al final de este paso, deberías poder afirmar todo esto como verdadero:

| Validación                                                                                 | Estado esperado |
|--------------------------------------------------------------------------------------------|-----------------|
| El binario activo de `git` viene de Nix (`~/.nix-profile/bin/git` o `/nix/store/.../git`)  | ✅              |
| `git --version` devuelve la versión de Nix, no “Apple Git-154”                             | ✅              |
| `git config --global --list` ya incluye tu nombre, correo y rama `main`                    | ✅              |
| `~/.config/git/ignore` existe y Git lo está usando como `core.excludesFile`               | ✅              |
| `git diff` usa `delta` con columnas lado a lado, números de línea y colores                | ✅              |

Esto significa algo importante:
tu identidad como desarrollador ya está descrita en un archivo declarativo (`flake.nix`),
no en configuraciones manuales y frágiles.

Eres portable.

En otra Mac, aplicas la misma configuración y obtienes:
- tu Git,
- tu estilo de diff,
- tu nombre,
- tu editor,
- tu forma de trabajar.

Nada global.  
Todo declarativo, reproducible, portable, controlado por ti.

## 6.4 Neovim — Editor reproducible y personal declarativo

### Propósito narrativo  
El editor es donde tus ideas se vuelven código.  
En este punto no se trata solo de escribir, sino de crear un espacio de trabajo coherente con tu filosofía.  

> “Neovim es la extensión natural de tu mente técnica;  
> su configuración no se instala, se declara.”

---

### Propósito técnico  
El objetivo es instalar **Neovim** en modo declarativo usando **Home Manager**,  
dejando que los *toolchains* y LSPs se declaren más adelante a nivel proyecto.  
Tu editor siempre existirá, pero sus capacidades se adaptarán al contexto de cada `devShell`.

---

### Paso a paso

#### 1. Declarar Neovim en Home Manager

Edita tu `flake.nix` en `~/dev/hm/` y dentro del bloque principal agrega:

```nix
programs.neovim = {
  enable = true;
  viAlias = true;
  vimAlias = true;
  withNodeJs = true;
  withPython3 = true;
  withRuby = false;

  extraPackages = with pkgs; [
    ripgrep
    fd
    tree-sitter
    lazygit
  ];
};
```

Esto habilita Neovim y añade utilidades clave que lo complementan:
- `ripgrep`: búsqueda de texto ultra rápida  
- `fd`: búsqueda de archivos  
- `lazygit`: gestión visual de Git desde terminal  
- `tree-sitter`: resaltado y análisis de sintaxis avanzado  

---

#### 2. Establecer Neovim como editor por defecto

Asegura que las variables de entorno reflejen tu decisión declarativa:

```nix
home.sessionVariables = {
  EDITOR = "nvim";
  VISUAL = "nvim";
};
```

---

#### 3. Aplicar y validar instalación

Ejecuta:

```bash
nix run home-manager/master -- switch --flake ~/dev/hm#<usuario>
exec $SHELL -l
nvim --version | head -n 3
```

🖥️ **Salida esperada:**
```
NVIM v0.10.x
Build type: Release
...
```

Si el editor abre correctamente con `nvim`, estás trabajando con una versión declarada, reproducible y portátil.

---

#### Nota técnica — Solución al error `command not found` (rg, fd, lazygit, tree-sitter)

Durante la validación, puede ocurrir que los binarios de soporte (`ripgrep`, `fd`, `lazygit`, `tree-sitter`)  
no aparezcan en el PATH aunque estén declarados en `programs.neovim.extraPackages`.

📍 **Causa**  
En ciertas combinaciones de `nixpkgs` + `home-manager`, los paquetes definidos en  
`programs.neovim.extraPackages` se exponen únicamente al runtime interno de Neovim,  
pero no al perfil de usuario (`~/.nix-profile/bin`).

📍 **Síntomas**  
Ejecutar estos comandos produce errores:

```bash
which rg
rg --version
# salida: rg not found
```

📍 **Solución estable**  
Declarar estas herramientas también en el bloque `home.packages`,  
de modo que queden disponibles de forma global en tu sesión.

Ejemplo:

```nix
home.packages = with pkgs; [
  git
  git-lfs   # opcional

  # herramientas CLI disponibles siempre
  ripgrep
  fd
  tree-sitter
  lazygit
];
```

📍 **Resultado esperado tras aplicar y recargar sesión**

```bash
which rg
/Users/<usuario>/.nix-profile/bin/rg
rg --version
ripgrep 14.1.1
```

Esto confirma que los binarios provienen del Nix Store y que tu entorno  
ya incluye un toolbelt declarativo y reproducible.

**Interpretación narrativa**
> Este ajuste no rompe la filosofía, la refuerza.  
> No estás “instalando globalmente”; estás declarando tus herramientas base,  
> las que te acompañan en cualquier proyecto.  
> Así, tu editor no solo existe dentro de Nix, sino contigo.
