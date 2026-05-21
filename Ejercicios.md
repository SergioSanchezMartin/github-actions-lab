# Laboratorio GitHub Actions

En este laboratorio se practican conceptos básicos de **CI/CD con GitHub Actions**.

Se trabajará sobre el proyecto **Hangman** utilizado en clase.

Se crear workflows para:

- Hacer la build de un proyecto
- Ejecutar tests unitarios
- Construir imágenes Docker
- Publicar imágenes Docker en un registry
- Ejecutar tests end‑to‑end

## Entrega del laboratorio

Este laboratorio se entrega a través del siguiente **repositorio público en GitHub:**.

```
https://github.com/SergioSanchezMartin/github-actions-lab
```

El repositorio contiene:

```
.github/workflows/
<el código fuente proporcionado>
README.md
```

## Ejercicio 1. Workflow CI para el proyecto de frontend

**En este ejercicio se trabaja sobre el proyecto de [frontend](./code/hangman-front/).**

**El objetivo es crear un nuevo workflow que se dispare cuando haya cambios en el proyecto `hangman-front` y exista una nueva pull request (deben darse las dos condiciones a la vez). El workflow ejecutará las siguientes operaciones:**
- **Build del proyecto**
- **Ejecución de los test unitarios**

---

### Estructura necesaria para GitHub Actions

Para realizar este ejercicio voy a trabajar en una nueva rama llamada **`ci-front-workflow`**. En ella se añadirá un workflow de **GitHub Actions** encargado de comprobar automáticamente que el proyecto de frontend funciona correctamente antes de integrar cambios en la rama principal.

![Captura 1](./capturas/ejercicio01/captura-01.png)

Los workflows de GitHub Actions deben almacenarse dentro del repositorio en la siguiente ruta:

**`.github/workflows/`**

Por tanto, lo primero será crear esta estructura de carpetas en la raíz del proyecto. Dentro de esa carpeta se guardarán los ficheros de configuración de los workflows, normalmente con extensión **`.yml`** o **`.yaml`**.

En este caso, como vamos a crear un workflow de integración continua, el fichero se llamará:

**`ci-front.yaml`**

---

### Configuración del fichero `ci-front.yaml`

#### Nombre del workflow

El primer paso consiste en asignar un nombre descriptivo al workflow. En este caso se utilizará el nombre:

**`CI-front`**

```yaml
name: CI-front
```

---

### Evento que activa el workflow

A continuación se configura la sección **`on`**, que indica en qué situaciones debe ejecutarse el workflow.

Como el ejercicio pide que se lance cuando haya cambios en el proyecto hangman-front y exista una pull request sobre la rama **`main`**, se utilizarán los eventos **`pull_request`** y **`push`** junto con los filtros **`branches`** y **`paths`**.

De esta forma, GitHub Actions solo ejecutará el workflow cuando la pull request incluya modificaciones dentro de esa carpeta:

La configuración quedaría reflejada como se muestra a continuación:

```yaml
on:
  pull_request:
    branches: [ main ]
    paths: [ 'code/hangman-front/**' ]
  # Añadimos el caso push
  push:
    branches: [ main ]
    paths: [ 'code/hangman-front/**' ]
```

---

### Definición de los jobs

La sección **`jobs`** permite indicar las tareas que se van a ejecutar dentro del workflow.

En este ejercicio se necesitan dos trabajos diferentes:

- **`build`**, encargado de construir el proyecto.
- **`test`**, encargado de ejecutar los test unitarios.

---

### Job `build`

El primer job se encargará de compilar o construir el proyecto de frontend.

En primer lugar, se indica el sistema operativo sobre el que se ejecutará el workflow. Para ello se utiliza la propiedad **`runs-on`** con el valor:

```yaml
runs-on: ubuntu-latest
```

Esto significa que GitHub Actions usará una máquina virtual con la última versión disponible de Ubuntu.

Después se definen los pasos dentro de la sección **`steps`**.

---

#### Paso 1: recuperar el código del repositorio

El primer paso consiste en descargar el código fuente del repositorio dentro de la máquina virtual donde se ejecuta el workflow.

Para ello se usa la acción oficial:

```yaml
actions/checkout@v6
```

El paso quedaría configurado así:
```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout
      uses: actions/checkout@v6
```

Esta acción permite que el workflow pueda acceder a los ficheros del proyecto.

---

#### Paso 2: configurar Node.js

El proyecto necesita Node.js para instalar dependencias, construir la aplicación y ejecutar los test.

Por ese motivo, el siguiente paso consiste en preparar la versión de Node.js que se va a utilizar. Para ello se emplea la acción:

En este caso se configurará Node.js en su versión 18:

```yaml
- name: Setup Node.js version
  uses: actions/setup-node@v6
  with:
    node-version: 18
```

---

#### Paso 3: construir el proyecto

Una vez disponible el código y configurado Node.js, se ejecuta la construcción del proyecto.

Como el fichero `package.json` no se encuentra en la raíz del repositorio, sino dentro de la carpeta **`hangman-front`**, es necesario indicar el directorio de trabajo mediante:

```yaml
- name: Build
  working-directory: ./code/hangman-front
```

Dentro de ese directorio se ejecutarán los comandos necesarios:

1. Instalar las dependencias.
2. Lanzar el script de build.

El paso quedaría como se muestra a continuación:

```yaml
- name: Build
    working-directory: ./code/hangman-front
    run: |
      npm ci
      npm run build --if-present
```

Con esto quedaría terminado el job **`build`**.

---

### Job `test`

El segundo job se encargará de ejecutar los test unitarios del proyecto.

Este job también se ejecutará sobre una máquina Ubuntu 22.04:

```yaml
runs-on: ubuntu-22.04
```

Además, interesa que los test se ejecuten después de que el proyecto se haya construido correctamente. Para establecer esa dependencia se utiliza:

```yaml
needs: build
```

De esta manera, el job **`test`** no comenzará hasta que el job **`build`** haya finalizado sin errores.

---

### Pasos del job `test`

Este job tendrá una estructura muy parecida al job anterior.

Los dos primeros pasos serán los mismos:

1. Recuperar el código del repositorio con **`actions/checkout@v6`**.
2. Configurar Node.js con **`actions/setup-node@v6`**.

El tercer paso será diferente, ya que en lugar de construir el proyecto se ejecutarán los test unitarios.

Para ello, de nuevo se indica como directorio de trabajo. A continuación se instalan las dependencias y se ejecuta el comando correspondiente a los test.

El paso final quedaría así:

```yaml
- name: Unit tests
        working-directory: ./code/hangman-front
        run: |
          npm ci
          npm run test
```

---

### Resultado final del workflow

Una vez configurados todos los apartados anteriores, el fichero **`ci-front.yaml`** contendrá el workflow completo de integración continua.

Su aspecto final será similar al siguiente:

```yaml
name: CI-front

on:
  pull_request:
    branches: [ main ]
    paths: [ 'code/hangman-front/**' ]
  push:
    branches: [ main ]
    paths: [ 'code/hangman-front/**' ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout
      uses: actions/checkout@v6
    - name: Setup Node.js version
      uses: actions/setup-node@v6
      with:
        node-version: 18
    - name: Build
      working-directory: ./code/hangman-front
      run: |
        npm ci
        npm run build --if-present

  test:
    runs-on: ubuntu-22.04
    needs: build

    steps:
      - name: Checkout
        uses: actions/checkout@v6
      - name: Setup Node.js version
        uses: actions/setup-node@v6
        with:
          node-version: 18
      - name: Unit tests
        working-directory: ./code/hangman-front
        run: |
          npm ci
          npm run test
```

---

### Comprobación del workflow

Para verificar que el workflow funciona correctamente, se realiza un cambio dentro del proyecto **`hangman-front`**.

Por ejemplo, se puede modificar temporalmente un fichero como **`app.tsx`**, añadiendo un pequeño comentario. Después se suben los cambios al repositorio mediante un **`push`**.

![Captura 2](./capturas/ejercicio01/captura-02.png)

![Captura 3](./capturas/ejercicio01/captura-03.png)

![Captura 4](./capturas/ejercicio01/captura-04.png)

Una vez subidos los cambios, GitHub muestra las diferencias entre la rama actual y la rama **`main`**.

![Captura 5](./capturas/ejercicio01/captura-05.png)

A continuación se pulsa el botón **`Compare & pull request`** para crear la pull request.

![Captura 6](./capturas/ejercicio01/captura-06.png)

![Captura 7](./capturas/ejercicio01/captura-07.png)

Como la pull request contiene cambios dentro de **`hangman-front`**, se cumplen las condiciones necesarias y el workflow se ejecuta automáticamente.

Desde la pestaña **Actions** se puede consultar el estado de la ejecución.

![Captura 8](./capturas/ejercicio01/captura-08.png)

---

### Revisión de errores

Durante la primera ejecución se observa que el job de test ha fallado. En este caso, espera un array de 1 posición pero se reciben 2.

![Captura 9](./capturas/ejercicio01/captura-09.png)

Para corregirlo, se revisa el error indicado por GitHub Actions y se realizan los cambios necesarios desde Visual Studio Code.

```tsx
expect(items).toHaveLength(2);
```

Además, se elimina el comentario temporal que se había añadido para provocar el cambio inicial.

![Captura 10](./capturas/ejercicio01/captura-10.png)

Después se vuelven a subir los cambios al repositorio.

![Captura 11](./capturas/ejercicio01/captura-11.png)

Al actualizarse la pull request, GitHub Actions ejecuta de nuevo el workflow. En esta segunda ejecución se comprueba que el proceso finaliza correctamente.

![Captura 12](./capturas/ejercicio01/captura-12.png)

---

### Finalización de la pull request

Una vez comprobado que el workflow se ejecuta sin errores, se puede hacer el **merge** de la pull request.

![Captura 13](./capturas/ejercicio01/captura-13.png)

En este caso, la rama no se elimina para que pueda consultarse posteriormente en el repositorio.

Por último, se vuelve a la rama **`main`** en el entorno local y se ejecuta un **`pull`** para actualizar el proyecto y dejarlo preparado para el siguiente ejercicio.



## 2. Workflow CD para el proyecto de frontend

**El objetivo es crear un nuevo workflow que se dispare manualmente y haga lo siguiente:**

- **Crear una nueva imagen de Docker**
- **Publicar dicha imagen en el [container registry de GitHub](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry)**

> **Nota: se usan las actions de Docker vistas en clase**

## 3. Workflow para ejecutar tests E2E (opcional)

**El objetivo es crear un workflow que se lance de la manera que elijamos y ejecute los tests e2e que encontrarás en [este enlace](https://github.com/Lemoncode/bootcamp-devops-lemoncode/tree/master/03-cd/03-github-actions/.start-code/hangman-e2e/e2e). Se puede usar [Docker Compose](https://docs.docker.com/compose/gettingstarted/) o [Cypress action](https://github.com/cypress-io/github-action) para ejecutar los tests.**
