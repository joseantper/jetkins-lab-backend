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