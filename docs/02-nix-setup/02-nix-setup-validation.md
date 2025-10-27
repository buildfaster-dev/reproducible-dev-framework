---
title: "Nix Setup Validation — macOS Tahoe 26.0.1"
version: "1.4"
author: "Jyr"
date: "2025-10-18"
description: "Validación paso a paso del documento 02-nix-setup-story.md. Incluye estructura estándar de workspace y registro completo con tee."
framework: "Reproducible Dev Framework"
---

# Nix Setup Validation — macOS Tahoe 26.0.1

> Este documento valida **paso a paso** cada sección de `02-nix-setup-story.md`.  
> Cada comando envía su salida a `~/nix-setup-validation.log` para mantener trazabilidad.

---

## 0️⃣ Preparar workspace estándar

```bash
mkdir -p ~/dev/{lab,personal,work,hm} 2>&1 | tee -a ~/nix-setup-validation.log
ls -1 ~/dev 2>&1 | tee -a ~/nix-setup-validation.log
```

**Salida esperada (ejemplo):**
```
hm
lab
personal
work
```
> A partir de aquí, todos los ejemplos y demos se crean bajo `~/dev/lab/`,  
> y la configuración de **Home Manager** vive en `~/dev/hm`.

---

## 1️⃣ Estado inicial del sistema

```bash
sw_vers 2>&1 | tee -a ~/nix-setup-validation.log
uname -m 2>&1 | tee -a ~/nix-setup-validation.log
which nix 2>&1 | tee -a ~/nix-setup-validation.log
ls /nix 2>&1 | tee -a ~/nix-setup-validation.log || echo "/nix no existe" | tee -a ~/nix-setup-validation.log
sudo launchctl list | grep nix 2>&1 | tee -a ~/nix-setup-validation.log || echo "nix-daemon no activo" | tee -a ~/nix-setup-validation.log
```

---

## 2️⃣ Instalación de Nix (multiusuario)

```bash
sh <(curl -L https://nixos.org/nix/install) --daemon 2>&1 | tee -a ~/nix-setup-validation.log
```

Si aparece error de respaldo:

```bash
sudo mv /etc/zshrc.backup-before-nix /etc/zshrc.backup-before-nix.bak 2>&1 | tee -a ~/nix-setup-validation.log
sudo mv /etc/bashrc.backup-before-nix /etc/bashrc.backup-before-nix.bak 2>&1 | tee -a ~/nix-setup-validation.log
```

Recargar sesión:

```bash
exec $SHELL -l
```

---

## 3️⃣ Verificar instalación inicial

```bash
nix --version 2>&1 | tee -a ~/nix-setup-validation.log
nix config check 2>&1 | tee -a ~/nix-setup-validation.log
```

Si obtienes el error:
```
error: experimental Nix feature 'nix-command' is disabled
```
continúa con la habilitación de features experimentales.

---

## 4️⃣ Habilitar features experimentales

```bash
sudo mkdir -p /etc/nix 2>&1 | tee -a ~/nix-setup-validation.log
echo "experimental-features = nix-command flakes fetch-tree" | sudo tee /etc/nix/nix.conf 2>&1 | tee -a ~/nix-setup-validation.log
sudo launchctl kickstart -k system/org.nixos.nix-daemon 2>&1 | tee -a ~/nix-setup-validation.log
exec $SHELL -l
```

Validación general:

```bash
nix config show | grep '^experimental-features' 2>&1 | tee -a ~/nix-setup-validation.log
```

Validación granular:

```bash
nix config show | grep -q 'nix-command' && echo "✔ nix-command OK" || echo "✘ falta nix-command" | tee -a ~/nix-setup-validation.log
nix config show | grep -q 'flakes' && echo "✔ flakes OK" || echo "✘ falta flakes" | tee -a ~/nix-setup-validation.log
nix config show | grep -q 'fetch-tree' && echo "✔ fetch-tree OK" || echo "⚠ fetch-tree no listado (opcional)" | tee -a ~/nix-setup-validation.log
```

---

## 5️⃣ Integrar Direnv + Flake minimal

> **Objetivo de validación**  
> Confirmar que el entorno declarativo básico responde al contexto del directorio y se activa manualmente con `nix develop`.  
> **Aún no** usamos `direnv` (se declarará con Home Manager en un paso posterior).

---

### 5.1 Crear el entorno y archivos iniciales

```bash
mkdir -p ~/dev/lab/direnv-demo 2>&1 | tee -a ~/nix-setup-validation.log
cd ~/dev/lab/direnv-demo 2>&1 | tee -a ~/nix-setup-validation.log
printf "use flake\n" > .envrc
ls -la 2>&1 | tee -a ~/nix-setup-validation.log
```

🖥️ **Salida esperada (contiene al menos):**
```
.envrc
```

---

### 5.2 Definir flake minimal y validar

```bash
cat > flake.nix <<'EOF'
{
  description = "Primer entorno reproducible con direnv y Nix (minimal)";
  inputs.nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";

  outputs = { nixpkgs, ... }: {
    devShells.aarch64-darwin.default = let
      pkgs = import nixpkgs { system = "aarch64-darwin"; };
    in pkgs.mkShell {
      buildInputs = [ pkgs.hello ];
      shellHook = ''
        echo "✅ Entorno reproducible activo"
      '';
    };
  };
}
EOF
```

```bash
nix flake check 2>&1 | tee -a ~/nix-setup-validation.log
```

> Esta validación crea/actualiza `flake.lock` y confirma que el flake es evaluable.

---

### 5.3 Activar el entorno manualmente y validar binario

```bash
nix develop 2>&1 | tee -a ~/nix-setup-validation.log
which hello 2>&1 | tee -a ~/nix-setup-validation.log
hello 2>&1 | tee -a ~/nix-setup-validation.log
```

🖥️ **Resultados esperados:**
```
/nix/store/...-hello-<versión>/bin/hello
Hello, world!
```

> El mensaje “✅ Entorno reproducible activo” se imprime desde el `shellHook`.

---

### 5.4 (Nota) Sobre `direnv` en esta etapa

```bash
command -v direnv 2>/dev/null || echo "OK: direnv aún no está instalado (se declarará con Home Manager)"
```

📝 **Expectativa correcta:**  
- **No** ejecutamos `direnv allow/status` todavía.  
- La autoactivación al hacer `cd` ocurrirá cuando declares `direnv` vía **Home Manager** en el paso correspondiente.  
- Este paso solo valida que tu **flake minimal** funciona y el entorno se activa manualmente con `nix develop`.

---

### 5.5 Validaciones rápidas

| Comando           | Propósito                           | Resultado esperado                                  |
|-------------------|-------------------------------------|-----------------------------------------------------|
| `nix develop`     | Activación manual del devShell      | “✅ Entorno reproducible activo”                    |
| `which hello`     | Verificar binario reproducible      | Ruta en `/nix/store/.../bin/hello`                  |
| `command -v direnv` | (Informativo) presencia de direnv | Puede estar ausente en esta etapa; es **correcto**  |
| `nix develop -c direnv status`         | Estado del hook                     | `.envrc` reconocido (`Found RC allowed 0`)          |

---

✅ **Resultado final**  
El entorno **minimal** responde correctamente y puede activarse manualmente con `nix develop`.  El entorno responde al contexto del directorio; `direnv` funciona desde el devShell,  
y la activación manual es correcta. La integración y autoactivación con `direnv` se validará más adelante cuando se declare con **Home Manager**.



## 6️⃣ Declarar Home Manager (nivel usuario)

> **Objetivo de validación**  
> Declarar Home Manager como la capa de usuario responsable de instalar y configurar `direnv` de manera declarativa.  
> Este paso convierte tu entorno personal en algo reproducible, versionable y controlado por Nix.

---

### 6.1 Contexto previo

Hasta este punto, has trabajado desde tu laboratorio (`~/dev/lab`)  
creando proyectos aislados con `flake.nix` y `nix develop`.  
Ahora darás un salto de nivel:  
crearás la capa **Home Manager**, donde tu configuración personal —no tus proyectos— vivirá.

📍 **Ubicación correcta antes de continuar:**
```bash
cd ~/dev
mkdir -p hm
```

🖋️ **Estructura esperada:**
```
~/dev/
├── hm/          ← Aquí irá tu flake de Home Manager
└── lab/         ← Aquí viven tus proyectos (como direnv-demo)
```

---

### 6.2 Crear el flake de Home Manager

```bash
echo "### 6.2 Crear flake de Home Manager" | tee -a ~/nix-setup-validation.log
USER_NAME="$(whoami)"
cd ~/dev/hm 2>&1 | tee -a ~/nix-setup-validation.log

cat > flake.nix <<EOF
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
    homeConfigurations.${USER_NAME} = home-manager.lib.homeManagerConfiguration {
      inherit pkgs;
      modules = [
        {
          home.username = "${USER_NAME}";
          home.homeDirectory = "/Users/${USER_NAME}";

          programs.zsh.enable = true;

          programs.direnv = {
            enable = true;
            nix-direnv.enable = true;
          };

          home.stateVersion = "24.05";
        }
      ];
    };
  };
}
EOF

ls -la ~/dev/hm 2>&1 | tee -a ~/nix-setup-validation.log
```

🖥️ **Salida esperada:**
```
flake.nix
```

---

### 6.3 Aplicar la configuración con Home Manager

```bash
echo "### 6.3 Aplicar configuración declarativa de HM" | tee -a ~/nix-setup-validation.log
nix run home-manager/master -- switch --flake ~/dev/hm#${USER_NAME} 2>&1 | tee -a ~/nix-setup-validation.log
```

🧠 **Qué ocurre aquí:**
1. Home Manager se instala (si no existía).  
2. Se crea tu perfil declarativo de usuario.  
3. Se instala `direnv` y su hook en `zsh`.

🖥️ **Salida esperada (resumen):**
```
Activating checkFilesChanged
Activating installPackages
replacing old 'home-manager-path'
installing 'home-manager-path'
Activating linkGeneration
Activating setupLaunchAgents
```

---

### 6.4 Recargar la sesión del shell

```bash
echo "### 6.4 Recargar sesión de shell" | tee -a ~/nix-setup-validation.log
exec $SHELL -l
```

> Este paso hace visible el hook de `direnv` en tu sesión actual.  
> No es necesario editar `.zshrc` ni tocar PATHs manualmente.

---

### 6.5 Validar instalación de `direnv` (Home Manager)

```bash
echo "### 6.5 Validar direnv" | tee -a ~/nix-setup-validation.log
which direnv 2>&1 | tee -a ~/nix-setup-validation.log
direnv version 2>&1 | tee -a ~/nix-setup-validation.log
```

🖥️ **Salida esperada:**
```
/nix/store/.../bin/direnv
direnv v2.x.x
```

---

### 6.6 Autorizar el proyecto reproducible y confirmar activación

```bash
echo "### 6.6 Autorizar proyecto direnv-demo" | tee -a ~/nix-setup-validation.log
cd ~/dev/lab/direnv-demo 2>&1 | tee -a ~/nix-setup-validation.log
direnv allow 2>&1 | tee -a ~/nix-setup-validation.log
direnv status 2>&1 | tee -a ~/nix-setup-validation.log
```

🖥️ **Salida esperada:**
```
Found RC allowed true
```

> Si aparece `Found RC allowed 0`, ejecuta nuevamente `direnv allow` dentro del mismo directorio.

---

### 6.7 Validar autoactivación del entorno

```bash
echo "### 6.7 Validar autoactivación" | tee -a ~/nix-setup-validation.log
cd ~ 2>&1 | tee -a ~/nix-setup-validation.log
cd ~/dev/lab/direnv-demo 2>&1 | tee -a ~/nix-setup-validation.log
hello 2>&1 | tee -a ~/nix-setup-validation.log
```

🖥️ **Salida esperada:**
```
direnv: loading ~/dev/lab/direnv-demo/.envrc
✅ Entorno reproducible activo
Hello, world!
```

> Esto demuestra que la activación ya no depende de `nix develop`:  
> el hook de Home Manager lo hace por ti.

---

### 6.8 Validaciones rápidas

| Comando | Propósito | Resultado esperado |
|----------|------------|--------------------|
| `which direnv` | Verifica el binario instalado por HM | `/nix/store/.../bin/direnv` |
| `direnv status` | Revisa el estado del `.envrc` | `Found RC allowed true` |
| `cd ~/dev/lab/direnv-demo` | Simula autoactivación | Mensaje del `shellHook` activo |
| `hello` | Verifica el entorno reproducible | `Hello, world!` |

---

### 6.9 Troubleshooting

| Síntoma | Causa probable | Solución | Validación |
|----------|----------------|-----------|-------------|
| `direnv: command not found` | Home Manager no aplicado | Repite `nix run home-manager/master -- switch …` y recarga la shell | `which direnv` |
| `Found RC allowed 0` | `.envrc` no autorizado aún | Ejecuta `direnv allow` dentro del proyecto | `direnv status` |
| No se autoactiva el entorno | El hook de zsh no está cargado | Abre una nueva ventana de terminal | Mensaje “✅ Entorno reproducible activo” |
| `hello: command not found` | El `devShell` no se activó | Verifica `.envrc` y `flake.nix` | `nix flake check` sin errores |

---

✅ **Resultado final**

Tu entorno de usuario ahora es **declarativo y reproducible**.  
`Home Manager` gestiona tus herramientas personales,  
y cada vez que entras a un proyecto en `~/dev/lab`,  
el entorno se autoactiva sin que tengas que ejecutar nada.

> “Nada global. Todo declarativo, reproducible, portable, controlado por ti.”

---

## 7. Validación — Git Declarativo (Home Manager)

### 🧩 Objetivo de la validación
Verificar que **Git** se haya instalado correctamente y esté gestionado de forma **declarativa** a través de **Home Manager**, priorizando el binario de Nix sobre el del sistema y confirmando que la configuración global y los archivos gestionados estén activos.

---

### 7.1 Verificar binario activo y PATH

```bash
which -a git | tee -a ~/nix-setup-validation.log
git --version | tee -a ~/nix-setup-validation.log
```

🖥️ **Salida esperada:**
```
/Users/<usuario>/.nix-profile/bin/git
/usr/bin/git
git version 2.51.0
```

> Si `/usr/bin/git` aparece primero o si la versión es “Apple Git-154”, revisa el bloque `home.sessionPath` y `programs.zsh.initContent` en tu flake de Home Manager.

---

### 7.2 Confirmar configuración global declarativa

```bash
git config --global --list | sort | tee -a ~/nix-setup-validation.log
```

🖥️ **Salida esperada (ejemplo resumido):**
```
user.name=Tu Nombre
user.email=tu.email@dominio.com
init.defaultbranch=main
pull.ff=only
push.default=simple
credential.helper=osxkeychain
core.editor=nvim
core.excludesfile=/Users/<usuario>/.config/git/ignore
```

✅ Confirma que:
- `user.name` y `user.email` sean los declarados.  
- `core.editor=nvim` si lo configuraste en tu flake.  
- `core.excludesfile` apunte a `~/.config/git/ignore`.

---

### 7.3 Validar archivo global de ignore gestionado por Home Manager

```bash
git config --global --get core.excludesFile | tee -a ~/nix-setup-validation.log
test -f ~/.config/git/ignore && echo "✅ Ignore file present" || echo "❌ Missing ignore file" | tee -a ~/nix-setup-validation.log
```

🖥️ **Salida esperada:**
```
/Users/<usuario>/.config/git/ignore
✅ Ignore file present
```

El contenido de ese archivo debe ser el generado por Home Manager (sin errores de sintaxis).  
Puedes revisarlo con:

```bash
head -n 10 ~/.config/git/ignore
```

---

### 7.4 Validar integración de Delta (diffs mejorados)

```bash
delta --version | tee -a ~/nix-setup-validation.log
git diff --color | head -n 20 | tee -a ~/nix-setup-validation.log
```

🖥️ **Salida esperada (ejemplo):**
```
delta 0.17.x
# Diferencias coloreadas con side-by-side activado
```

Si `delta` no está disponible, verifica que esté incluido en `home.packages` o dentro del bloque `programs.delta`.

---

### 7.5 Confirmar alias de Git activos

```bash
git config --global --get-regexp alias | tee -a ~/nix-setup-validation.log
```

🖥️ **Salida esperada (ejemplo):**
```
alias.co checkout
alias.br branch
alias.st status -sb
alias.ci commit
alias.ca commit --amend
alias.lg log --graph --decorate --oneline --all
alias.undo reset --soft HEAD~1
```

> Si los alias no aparecen, revisa que el bloque de `programs.git.settings.alias` esté correctamente definido y sin errores de indentación.

---

### 7.6 Validar persistencia tras reinicio de sesión

Cierra la sesión de tu terminal y ábrela nuevamente (WezTerm o iTerm).  
Luego ejecuta:

```bash
which -a git | tee -a ~/nix-setup-validation.log
git --version | tee -a ~/nix-setup-validation.log
```

🖥️ **Salida esperada:**
```
/Users/<usuario>/.nix-profile/bin/git
/usr/bin/git
git version 2.51.0
```

Esto confirma que Home Manager mantiene el PATH y el binario de Nix como prioridad incluso tras reiniciar la sesión.

---

### 7.7 Validar que no existan configuraciones heredadas conflictivas

```bash
git config --global --list --show-origin | grep -i include | tee -a ~/nix-setup-validation.log
```

🖥️ **Salida esperada:**
```
# (sin resultados)
```

Si aparece algo como `include.path=~/.config/git/ignore`, elimínalo con:

```bash
git config --global --unset-all include.path
```

---

### Resultado esperado

| Validación | Estado | Descripción |
|-------------|---------|-------------|
| Binario Git de Nix activo (`which -a git`) | ✅ | Prioriza el binario declarativo sobre Apple Git. |
| Configuración global aplicada (`user.name`, `editor`, `branch`) | ✅ | Refleja los valores definidos en Home Manager. |
| Archivo de ignore gestionado por HM | ✅ | Existe y está correctamente vinculado. |
| Delta activo y diffs coloreados | ✅ | Mejoras visuales aplicadas. |
| Alias funcionales (`git lg`, `git undo`, etc.) | ✅ | Alias disponibles sin configuración manual. |
| PATH persistente tras reinicio | ✅ | Home Manager mantiene el entorno reproducible. |
| Sin includes heredados | ✅ | No existen conflictos ni configuraciones previas. |

---

> **Conclusión:**  
> Git ahora forma parte de tu capa personal declarativa.  
> Cada commit, diff o alias que ejecutes está respaldado por Nix, sin configuraciones globales manuales.  
> “Nada global. Todo declarativo, reproducible, portable, controlado por ti.”

---
## 8. Validación — Neovim Declarativo (Home Manager)

📍 **Objetivo**  
Confirmar que Neovim, junto con sus herramientas de soporte (`ripgrep`, `fd`, `lazygit`, `tree-sitter`),  
se encuentra instalado y disponible dentro del entorno declarativo gestionado por Home Manager.

---

### 1. Verificar versiones y rutas

```bash
which nvim | tee -a ~/nix-setup-validation.log
nvim --version | head -n 3 | tee -a ~/nix-setup-validation.log

which rg | tee -a ~/nix-setup-validation.log
rg --version | tee -a ~/nix-setup-validation.log

which fd | tee -a ~/nix-setup-validation.log
fd --version | tee -a ~/nix-setup-validation.log

which lazygit | tee -a ~/nix-setup-validation.log
lazygit --version | tee -a ~/nix-setup-validation.log || true
```

🖥️ **Salida esperada (ejemplo):**

```
/Users/<usuario>/.nix-profile/bin/nvim
NVIM v0.10.x

/Users/<usuario>/.nix-profile/bin/rg
ripgrep 14.1.1

/Users/<usuario>/.nix-profile/bin/fd
fd 10.3.0

/Users/<usuario>/.nix-profile/bin/lazygit
commit=, build date=, build source=nix, version=0.55.1, os=darwin, arch=arm64, git version=2.51.0
```

**Interpretación**
- Los binarios provienen de `~/.nix-profile/bin`, no de `/usr/bin`.
- Cada herramienta fue declarada en `flake.nix`.
- Todo se ejecuta desde Nix Store (`/nix/store/...`).

---

#### Validación adicional — Corrección del error `not found` (rg, fd, lazygit, tree-sitter)

Durante la validación inicial se detectó que las herramientas de soporte (`rg`, `fd`, `lazygit`, `tree-sitter`)  
no estaban presentes en el perfil del usuario (`~/.nix-profile/bin`), mostrando errores como:

```
zsh: command not found: rg
```

📍 **Diagnóstico**
- Los binarios se declararon únicamente en `programs.neovim.extraPackages`.
- En algunas versiones de Home Manager, esos paquetes **no se exportan al PATH global**.

📍 **Solución**
Añadir los mismos paquetes al bloque `home.packages` dentro del módulo de Home Manager, por ejemplo:

```nix
home.packages = with pkgs; [
  git
  git-lfs
  ripgrep
  fd
  tree-sitter
  lazygit
];
```

📍 **Validación tras aplicar Home Manager y recargar sesión**

```bash
ls -l ~/.nix-profile/bin/{rg,fd,lazygit,tree-sitter} | tee -a ~/nix-setup-validation.log
which rg | tee -a ~/nix-setup-validation.log
which fd | tee -a ~/nix-setup-validation.log
which lazygit | tee -a ~/nix-setup-validation.log
```

🖥️ **Salida esperada (ejemplo real validado):**
```
/Users/<usuario>/.nix-profile/bin/rg -> /nix/store/...-ripgrep-14.1.1/bin/rg
/Users/<usuario>/.nix-profile/bin/fd -> /nix/store/...-fd-10.3.0/bin/fd
/Users/<usuario>/.nix-profile/bin/lazygit -> /nix/store/...-lazygit-0.55.1/bin/lazygit
/Users/<usuario>/.nix-profile/bin/tree-sitter -> /nix/store/...-tree-sitter-0.25.6/bin/tree-sitter
```

 **Conclusión**
Los binarios ahora viven bajo el perfil declarativo (`~/.nix-profile/bin`),  
gestionados por Nix, y el entorno funciona conforme a los principios del sistema:

> Nada global.  
> Todo declarativo, reproducible, portable, controlado por ti.
