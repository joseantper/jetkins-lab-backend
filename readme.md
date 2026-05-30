# Practica Automatización CI/CD con Jenkinsfile sobre el proyecto backend
## Autor: José Antonio Pérez Alías

<br>
<br>

---



## 1. ¿Qué es Jenkins?
Jenkins es el servidor de automatización de código abierto más utilizado en la industria del DevOps. Diseñado inicialmente como herramienta de **Integración Continua (CI)**, ha evolucionado para cubrir flujos completos de **Entrega/Despliegue Continuo (CD)**. 

Su arquitectura se basa en un modelo **Controlador-Agente**:
- **Controlador (Controller):** Gestiona la configuración, orquesta los jobs, expone la interfaz web y las APIs REST.
- **Agentes (Workers):** Ejecutan las tareas reales (compilación, tests, despliegues), permitiendo escalabilidad horizontal y aislamiento por sistema operativo o stack tecnológico.

Jenkins destaca por su ecosistema de **plugins** y su motor de orquestación basado en **Groovy**, lo que permite definir pipelines como código, versionables, auditables y reproducibles.

## 2. Contexto de la Práctica: Pipeline as Code y MultiBranch
En la metodología moderna de DevOps, la configuración de los flujos de CI/CD no debe vivir en la UI de Jenkins, sino en el propio repositorio del proyecto. Esto se logra mediante un `Jenkinsfile`, que define el flujo usando la sintaxis **Declarative Pipeline**.

Al combinar `Jenkinsfile` con un **Proyecto MultiBranch**, Jenkins escanea automáticamente el repositorio de GitHub, detecta ramas (`branches`) y Pull Requests, y crea instancias independientes del pipeline para cada una, ejecutando siempre la versión del `Jenkinsfile` asociada a esa rama. Esto garantiza que el proceso de build esté siempre sincronizado con el código y aplica buenas prácticas de trazabilidad.

---


## 2. Copiar proyecto localmente
  ![Captura 1](./capturas/tarea-1-1.png) 


## 3. Implementación del `Jenkinsfile`
El siguiente código cumple estrictamente con todos los requisitos del enunciado. Debe ubicarse en la raíz del repositorio o dentro de `backend/` (ajustando la ruta en la configuración de Jenkins).

```groovy
pipeline {
    agent any

    options {
        disableConcurrentBuilds()
        timestamps()
        timeout(time: 5, unit: 'MINUTES')
    }

    environment {
        FORCE_COLOR = '0'
        NO_COLOR = 'true'
    }

    stages {
        stage('Audit tools') {
            steps {
                sh 'node --version'
            }
        }
        stage('Install dependencies') {
            steps {
                sh 'npm install'
            }
        }
        stage('Generate files') {
            steps {
                sh 'npm run prisma:generate'
            }
        }
        stage('Format check') {
            steps {
                sh 'npm run format:check'
            }
        }
        stage('Code quality') {
            steps {
                sh 'npm run lint'
            }
        }
        stage('Type check') {
            steps {
                sh 'npm run type-check'
            }
        }
        stage('Tests') {
            steps {
                sh 'npm run test'
            }
        }
        stage('Build') {
            steps {
                sh 'npm run build'
                archiveArtifacts artifacts: 'dist/**', fingerprint: true
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Review logs.'
        }
        always {
            cleanWs()
        }
    }
}
```

<br>

A continuación se muestra una infografia del código anterior comentada por bloques
![Captura](./capturas/tarea-1-2.png) 

- Lo siguiente será levantar **`Jenkins`** mediante el comando **`docker compose up`**:

![Captura](./capturas/tarea-1-3.png) 

- estar atento a la contraseña que nos proporciona durante el arranque

![Captura](./capturas/tarea-1-3.png) 

- una vez arrancados ya podemos conectarnos al servidor jenkins por el puerto 8080

![Captura](./capturas/tarea-1-4.png) 

- debemos introducir la contraseña que se generó al arrancar, ya estariamos dentro

![Captura](./capturas/tarea-1-5.png)

- seleccionamos la instalacion de plugins por defecto

![Captura](./capturas/tarea-1-6.png)

- una vez finalizado, nos va a solicitar crear el primer usuario

![Captura](./capturas/tarea-1-7.png)

- en nuestro caso como vamos a trabajar local lo podemos dejar como viene, en caso de usar un dominio, en este caso habria que indicar el dominio en este paso

![Captura](./capturas/tarea-1-8.png)

- ya estariamos dentro de jenkins

![Captura](./capturas/tarea-1-9.png)

- como ya tenenmos el fichero jenkisfile, lo que tenemos que enganchar jenkins con el repositorio

![Captura](./capturas/tarea-1-10.png)

- en nuestro caso escogeremos multibrach Pipeline

![Captura](./capturas/tarea-1-11.png)

* En el apartado de **Branch Sources** hacemos *clic* sobre **+ Add source** y seleccionamos un repositorio de **GitHub**:

![Captura](./capturas/tarea-1-12.png)

* copiamos la url de nuestro laboratorio https://github.com/joseantper/jetkins-lab-backend

![Captura](./capturas/tarea-1-13.png)

+ Para evitar bloqueos de **Github** crearemos una credencial, para ello:
            - Hacemos *clic* en nuestra foto del avatar de **Github**.
            - En menú que aparece seleccionamos **Settings**.
            - Al final del menú que aparece a la izquierda, seleccionamos **<> Developer settings**:

![Captura](./capturas/tarea-1-14.png)
![Captura](./capturas/tarea-1-15.png)

- En la ventana que aparece, desplegamos **Personal access tokens** y seleccionamos **Fine-grained tokens**.

![Captura](./capturas/tarea-1-16.png)
  
- Pulsamos sobre **Generate new token**.

![Captura](./capturas/tarea-1-17.png)
![Captura](./capturas/tarea-1-18.png)

- Indicamos un *nombre* para el *token*, para este caso: **`jenkins-backend-token`**.
            - En *Repository access* seleccionamos **Only select repositories** y, en el desplegable seleccionamos el repositorio, para este caso: **`/josedavid-quero/jenkins-backend-lab`**.
            - En *Permissions* pulsamos sobre *+ Add Permissions*:
                * Le damos permiso de **Contents**: *Read-only* para que quien use este *token* sea capaz de ver el contenido del repositorio .

![Captura](./capturas/tarea-1-19.png)
![Captura](./capturas/tarea-1-20.png)

* Le damos permiso de **Commit statuses**: *Read and write* para que cuando **`Jenkins`** termine una *pipeline* le envíe a **Github** automáticamente una petición indicándole que el *commit* sobre el que ha hecho la *build* está todo bien.

![Captura](./capturas/tarea-1-21.png)
![Captura](./capturas/tarea-1-22.png)

* te indica que va a generar un token para 1 repositorio con 3 permisos
![Captura](./capturas/tarea-1-23.png)

* Nos muestra que tenenmos el token generado, lo copiamos y volvemos a la pagina de jenkins
![Captura](./capturas/tarea-1-24.png)

* En el servidor Jenkins, damos a añadir credenciales
![Captura](./capturas/tarea-1-25.png)

* De las opciones que nos aparece, escogemos username with password

- Username: joseantper (nuestro usuario en github)
- Password: ponemos la cadena del token
- ID: nos lo inventamos, pero que sea único
- Description: un texto identificativo del credencial, por ejemplo Credenciales de GitHub.
- Pulsamos el botón Create.
![Captura](./capturas/tarea-1-26.png)

* Una vez creada volvemos a la página y seleccionamos la credencial creada
![Captura](./capturas/tarea-1-28.png)
![Captura](./capturas/tarea-1-27.png)

* Pues darle un CLIC FUERTE AL SAVE
se observa que ha arrancado, se conecta al repositorio y busca ramas del fichero jenkinksfile
![Captura](./capturas/tarea-1-29.png)

# ¿Qué esta pasando?

Pues que como le hemos dicho que se conecte a github, y el fichero todavia lo tenemos el local, **NO lo ENCUENTRA**, para que esto funcione debemos subir todo nuestro trabajo a github

![Captura](./capturas/tarea-1-30.png)

* Tenemos un problema, y nos toca revisar los logs. Lo que nos dice que no encuentra 
```
npm error enoent Could not read package.json: Error: ENOENT: no such file or directory, open '/var/jenkins_home/workspace/jenkins-backend-01_main/package.json'
```
El proyecto NO tiene el archivo package.json en la raíz del repositorio. Esta dentro de la carpeta llamada backend. Para dejarlo con la estructura más limpia, lo que haremos es modificar el Jenkinsfile, para indicarle donde estan los archivos

![Captura](./capturas/tarea-1-31.png)


Ahora si esta funcionando
![Captura](./capturas/tarea-1-32.png)
![Captura](./capturas/tarea-1-35.png)

Como no coincide con las graficas de mis compañeros, es porque me falta un plugin, 
Administrar Jenkins > Plugins > Plugins Disponibles, busca Pipeline Stage View e instálalo.

![Captura](./capturas/tarea-1-36.png)
![Captura](./capturas/tarea-1-37.png)

Ahora podemos ver nuestros SOLETE
![Captura](./capturas/tarea-1-38.png)

Y nuestra temporizacion
![Captura](./capturas/tarea-1-39.png)

POR FIN!!!!! ALELUYA
![Captura](./capturas/tarea-1-40.png)

<br>
<br>
<br>
<br>

---

# Explicacion PASO a PASO fichero Jenkinsfile

# Explicación del pipeline de Jenkins

Este documento explica, bloque por bloque, qué hace el pipeline de Jenkins del proyecto.

La idea es entender el flujo de trabajo de forma clara y sencilla: Jenkins prepara el entorno, instala dependencias, comprueba la calidad del código, ejecuta pruebas, compila el backend y finalmente limpia el espacio de trabajo.

---

## 1. Bloque principal del pipeline

```groovy
pipeline {
    agent any

    ...
}
```

Este es el bloque principal del fichero Jenkinsfile.

Todo lo que Jenkins debe ejecutar se define dentro de `pipeline { ... }`.

La línea:

```groovy
agent any
```

indica que Jenkins puede ejecutar este pipeline en cualquier agente disponible.

Un **agente** es la máquina o entorno donde Jenkins lanza los comandos. Al usar `any`, Jenkins elegirá automáticamente un nodo disponible para ejecutar el trabajo.

---

## 2. Opciones generales del pipeline

```groovy
options {
    disableConcurrentBuilds()
    timestamps()
    timeout(time: 5, unit: 'MINUTES')
}
```

Este bloque configura algunas opciones generales que afectan a todo el pipeline.

---

### 2.1. Evitar ejecuciones simultáneas

```groovy
disableConcurrentBuilds()
```

Esta opción evita que dos ejecuciones del mismo pipeline se lancen al mismo tiempo.

Es útil porque impide que dos builds trabajen sobre el mismo proyecto a la vez, lo que podría provocar errores o resultados mezclados.

Por ejemplo, evita problemas como:

- Dos builds usando el mismo workspace.
- Dos procesos instalando dependencias al mismo tiempo.
- Artefactos generados por una ejecución mezclados con los de otra.

En resumen: **una ejecución debe terminar antes de que empiece otra**.

---

### 2.2. Añadir hora a los logs

```groovy
timestamps()
```

Esta opción añade marcas de tiempo a la salida de consola de Jenkins.

Gracias a esto, en los logs se puede ver a qué hora se ejecutó cada comando.

Es útil para saber:

- Cuánto tarda cada fase.
- En qué momento ocurrió un error.
- Qué pasos consumen más tiempo.

---

### 2.3. Limitar el tiempo máximo de ejecución

```groovy
timeout(time: 5, unit: 'MINUTES')
```

Esta opción establece un tiempo máximo para ejecutar el pipeline completo.

En este caso, Jenkins detendrá el proceso si tarda más de **5 minutos**.

Esto evita que el job se quede bloqueado indefinidamente si algún comando se cuelga o no termina correctamente.

---

## 3. Variables de entorno

```groovy
environment {
    FORCE_COLOR = '0'
    NO_COLOR = 'true'
}
```

Este bloque define variables de entorno que estarán disponibles durante la ejecución del pipeline.

En este caso se usan para controlar los colores en la consola.

---

### 3.1. Variable `FORCE_COLOR`

```groovy
FORCE_COLOR = '0'
```

Esta variable indica que no se debe forzar el uso de colores en la salida de consola.

Algunas herramientas de Node.js muestran colores automáticamente. En Jenkins, esos colores pueden verse mal si la consola no los interpreta correctamente.

---

### 3.2. Variable `NO_COLOR`

```groovy
NO_COLOR = 'true'
```

Esta variable indica a muchas herramientas que no deben mostrar colores en la salida.

El objetivo es que los logs de Jenkins sean más limpios y fáciles de leer.

---

## 4. Bloque de etapas

```groovy
stages {
    ...
}
```

El bloque `stages` contiene todas las fases que Jenkins va a ejecutar.

Cada fase se define con un bloque `stage`.

Dividir el pipeline en etapas permite ver claramente en Jenkins:

- Qué parte se está ejecutando.
- Qué parte ha fallado.
- Cuánto tarda cada etapa.
- En qué punto se detuvo el proceso.

---

# Etapas del pipeline

---

## 5. Etapa `Audit tools`

```groovy
stage('Audit tools') {
    steps {
        sh 'node --version'
        sh 'npm --version'
    }
}
```

Esta etapa comprueba las versiones de las herramientas principales del proyecto.

Ejecuta estos comandos:

```bash
node --version
npm --version
```

Con esto Jenkins muestra en consola qué versión de **Node.js** y de **npm** está usando.

Es una comprobación sencilla, pero muy importante. Si Node.js o npm no estuvieran instalados correctamente, el pipeline fallaría desde el principio.

Esta fase sirve para verificar que el entorno de Jenkins está preparado antes de continuar.

---

## 6. Etapa `Install dependencies`

```groovy
stage('Install dependencies') {
    steps {
        dir('backend') {
            sh 'npm install'
        }
    }
}
```

Esta etapa instala las dependencias del backend.

El bloque:

```groovy
dir('backend') {
    ...
}
```

indica que los comandos se ejecutarán dentro de la carpeta `backend`.

Dentro de esa carpeta se ejecuta:

```bash
npm install
```

Este comando lee el fichero `package.json` y descarga las librerías necesarias para que el proyecto funcione.

En resumen, esta etapa prepara el proyecto instalando todo lo necesario antes de hacer comprobaciones, pruebas o compilación.

---

## 7. Etapa `Generate files`

```groovy
stage('Generate files') {
    steps {
        dir('backend') {
            sh 'npm run prisma:generate'
        }
    }
}
```

Esta etapa genera ficheros necesarios para el backend.

El comando que se ejecuta es:

```bash
npm run prisma:generate
```

En proyectos que usan **Prisma**, este comando suele generar el cliente de Prisma a partir del esquema de base de datos.

Dicho de forma sencilla: prepara el código que la aplicación necesita para comunicarse con la base de datos.

Esta etapa es importante porque otras fases, como la comprobación de tipos o la compilación, pueden necesitar esos ficheros generados.

---

## 8. Etapa `Format check`

```groovy
stage('Format check') {
    steps {
        dir('backend') {
            sh 'npm run format:check'
        }
    }
}
```

Esta etapa comprueba si el código está bien formateado.

Ejecuta:

```bash
npm run format:check
```

Normalmente, este comando se usa con herramientas como **Prettier**.

Su función no es cambiar el código, sino comprobar que el formato cumple las reglas definidas por el proyecto.

Por ejemplo, puede revisar:

- Espacios.
- Saltos de línea.
- Comillas.
- Sangrías.
- Estilo general del código.

Si el código no cumple el formato esperado, Jenkins marcará el pipeline como fallido.

---

## 9. Etapa `Code quality`

```groovy
stage('Code quality') {
    steps {
        dir('backend') {
            sh 'npm run lint'
        }
    }
}
```

Esta etapa revisa la calidad del código.

Ejecuta:

```bash
npm run lint
```

Normalmente, este comando usa herramientas como **ESLint**.

Sirve para detectar errores o malas prácticas en el código.

Por ejemplo, puede detectar:

- Variables declaradas pero no utilizadas.
- Imports incorrectos.
- Código mal estructurado.
- Reglas de estilo incumplidas.
- Posibles errores antes de ejecutar la aplicación.

Esta fase ayuda a mantener un código más limpio, seguro y fácil de mantener.

---

## 10. Etapa `Type check`

```groovy
stage('Type check') {
    steps {
        dir('backend') {
            sh 'npm run type-check'
        }
    }
}
```

Esta etapa comprueba los tipos del proyecto.

Ejecuta:

```bash
npm run type-check
```

En proyectos desarrollados con **TypeScript**, esta fase revisa que los tipos sean correctos.

Por ejemplo, puede detectar problemas como:

- Pasar un número donde se espera un texto.
- Usar una propiedad que no existe.
- Llamar a una función con parámetros incorrectos.
- Tener errores en interfaces o modelos.

Esta etapa permite encontrar errores antes de ejecutar o desplegar el backend.

---

## 11. Etapa `Tests`

```groovy
stage('Tests') {
    steps {
        dir('backend') {
            sh 'npm run test'
        }
    }
}
```

Esta etapa ejecuta las pruebas automáticas del backend.

El comando usado es:

```bash
npm run test
```

Las pruebas sirven para comprobar que el código funciona como se espera.

Dependiendo del proyecto, este comando puede ejecutar:

- Pruebas unitarias.
- Pruebas de integración.
- Pruebas de servicios.
- Pruebas de controladores.
- Pruebas de lógica de negocio.

Si alguna prueba falla, Jenkins detendrá el pipeline y lo marcará como fallido.

Esta fase es clave en DevOps porque ayuda a detectar errores antes de que lleguen a producción.

---

## 12. Etapa `Build`

```groovy
stage('Build') {
    steps {
        dir('backend') {
            sh 'npm run build'
            archiveArtifacts artifacts: 'dist/**', fingerprint: true
        }
    }
}
```

Esta etapa compila el backend y guarda el resultado generado.

Primero se ejecuta:

```bash
npm run build
```

Este comando construye la versión final del backend.

En muchos proyectos Node.js o TypeScript, el resultado de la compilación se guarda en una carpeta llamada `dist`.

Después se ejecuta:

```groovy
archiveArtifacts artifacts: 'dist/**', fingerprint: true
```

Este paso guarda los ficheros generados por la compilación como artefactos de Jenkins.

Como el comando está dentro de:

```groovy
dir('backend') {
    ...
}
```

el patrón `dist/**` se refiere a la carpeta `dist` que está dentro de `backend`.

---

### 12.1. ¿Qué hace `archiveArtifacts`?

```groovy
archiveArtifacts artifacts: 'dist/**', fingerprint: true
```

Este comando archiva los ficheros generados en la carpeta `dist`.

Eso permite que, al finalizar el pipeline, esos ficheros se puedan consultar o descargar desde Jenkins.

Es útil para conservar el resultado de la compilación.

---

### 12.2. ¿Qué significa `fingerprint: true`?

```groovy
fingerprint: true
```

Esta opción hace que Jenkins genere una huella identificativa de los artefactos.

Dicho de forma sencilla: Jenkins registra esos ficheros para poder rastrearlos.

Esto mejora la trazabilidad, porque permite saber de qué build salió cada artefacto.

---

# Acciones posteriores a la ejecución

---

## 13. Bloque `post`

```groovy
post {
    always {
        cleanWs()
    }
    success {
        echo 'Pipeline completed successfully!'
    }
    failure {
        echo 'Pipeline failed. Review logs.'
    }
}
```

El bloque `post` define acciones que se ejecutan al final del pipeline.

Estas acciones dependen del resultado de la ejecución.

En este caso hay tres situaciones:

- `always`: se ejecuta siempre.
- `success`: se ejecuta si todo ha ido bien.
- `failure`: se ejecuta si algo ha fallado.

---

## 14. Acción `always`

```groovy
always {
    cleanWs()
}
```

Esta acción se ejecuta siempre, tanto si el pipeline termina bien como si falla.

Dentro se ejecuta:

```groovy
cleanWs()
```

Este comando limpia el workspace de Jenkins.

El **workspace** es la carpeta donde Jenkins descarga el código y ejecuta los comandos.

Limpiar el workspace ayuda a evitar problemas en futuras ejecuciones.

Por ejemplo, evita que queden:

- Ficheros temporales.
- Dependencias antiguas.
- Artefactos de builds anteriores.
- Archivos generados que ya no deberían estar.

En resumen: deja el entorno limpio para la siguiente ejecución.

---

## 15. Acción `success`

```groovy
success {
    echo 'Pipeline completed successfully!'
}
```

Este bloque se ejecuta solamente si todas las etapas han terminado correctamente.

Muestra en la consola de Jenkins el mensaje:

```text
Pipeline completed successfully!
```

Sirve para indicar claramente que el pipeline ha finalizado con éxito.

---

## 16. Acción `failure`

```groovy
failure {
    echo 'Pipeline failed. Review logs.'
}
```

Este bloque se ejecuta si alguna etapa falla.

Muestra en la consola el mensaje:

```text
Pipeline failed. Review logs.
```

Indica que hay que revisar los logs de Jenkins para encontrar el error.

El fallo podría estar, por ejemplo, en:

- La instalación de dependencias.
- La generación de Prisma.
- El formateo del código.
- El análisis de calidad.
- La comprobación de tipos.
- Las pruebas.
- La compilación.

---

# Resumen sencillo del flujo

El pipeline ejecuta el siguiente proceso:

```text
1. Comprueba las versiones de Node.js y npm.
2. Entra en la carpeta backend.
3. Instala las dependencias del proyecto.
4. Genera los ficheros necesarios de Prisma.
5. Comprueba el formato del código.
6. Revisa la calidad del código.
7. Comprueba errores de tipos.
8. Ejecuta las pruebas automáticas.
9. Compila el backend.
10. Archiva los ficheros generados en dist.
11. Limpia el workspace.
12. Muestra un mensaje final según el resultado.
```

---

# Conclusión

Este pipeline automatiza un flujo básico de integración continua para un backend desarrollado con Node.js.

Su objetivo es comprobar que el proyecto está en buen estado antes de considerarlo válido.

Gracias a este pipeline, Jenkins puede detectar automáticamente errores de instalación, formato, calidad, tipos, pruebas o compilación.

Esto ayuda al equipo de desarrollo a trabajar de forma más segura, ordenada y profesional.


