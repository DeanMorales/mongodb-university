# connecting to a mongodb database using the mongodb shell

## conceptos clave

- connection string
    mongo prove 2 formatdos para las conecciones
- Standar formart
- SRV formato por defecto en atlas

- components

- estructura de una coneccion: 

consta de algunos campos obligatorios y varios campos opcionales, entre corchetes []

### pasos para optener Mongosh

Mongodb Shell (mongosh) es una shell poderoza para mongodb
- ofrece una interfaz para interactuar con nuestras bases de dato. 

#### en Ubuntu

tenemos que agregar el repositorio oficial de mongodb mediante los comandos 

```
sudo apt update
sudo apt install gnupg
wget -q0- "link-mongo-server"

```

ademas de que tenemos que tener a la mano nuestro enlace de coneccion, nuestro **coneection string**

y nuestras credenciales. para acceder a la base de datos. 

---

### laboratorio

en este pequeño laboratorio instalamos mongosh en una maquina virtual directamente en la plataforma de mongobd university

Install the MongoDB Shell (mongosh)
In this lab, you will install MongoDB Shell, also known as mongosh, in an Ubuntu environment.

First, you'll check that the required dependencies are installed. Then, you will create a list file for the version of Ubuntu that your container is using. Finally, you will update the local package index and install mongosh.

Note: As the user of this container, you have root privileges. This means commands that would normally require sudo are not necessary in this lab.

Lab Instructions
To install mongosh in an Ubuntu environment, you must have the required gnupg package installed on the system. The container for this lab has the required gnupg package installed. You can verify this by running the following command in the terminal tab:

bash

copy

run
gpg --version
Import the public key that's used by the package management system by running the following command:

bash

copy

run
wget -qO- https://www.mongodb.org/static/pgp/server-8.0.asc | tee /etc/apt/trusted.gpg.d/server-8.0.asc
Create a list file for the version of Ubuntu that's used in the container. A list file describes which URLs can be used to install software. It is used by the Advanced Package Tool (APT) to add new software repositories, enabling the package manager to know where to fetch the software it is being requested to install. To find out which version of Ubuntu is installed on the virtual machine, run the following command:

bash

copy

run
cat /etc/os-release
The output should look something like this:

bash
PRETTY_NAME="Ubuntu 24.04.4 LTS"
NAME="Ubuntu"
VERSION_ID="24.04"
VERSION="24.04.4 LTS (Noble Numbat)"
VERSION_CODENAME=noble
ID=ubuntu
ID_LIKE=debian
In this example, the version of Ubuntu is 24.04 that goes by the VERSION_CODENAME, noble. Using this information, create a list file for this version of Ubuntu by running the following command:

bash

copy

run
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu noble/mongodb-org/8.0 multiverse" | tee /etc/apt/sources.list.d/mongodb-org-8.0.list
This command creates a list file for Ubuntu 24.04. If you are using a different version of Ubuntu, then you will need to replace noble with the VERSION_CODENAME that's listed in the output of the cat /etc/os-release command.

Update your local package index and install mongosh. To do so, first run the following command to update the local package index:

bash

copy

run
apt update
Then, run the following command to install mongosh:

bash

copy

run
apt install -y mongodb-mongosh
Ensure that mongosh was installed successfully by running the following command:

bash

copy

run
mongosh --version
The output should look something like this:

bash
<MAJOR.MINOR.PATCH>
For example, if the installed version is 2.10.0, the output will be: 2.10.0

---

## troubleshooting conecctions errors

muchos de los problemas que se generan al conectarse, son debido a las restricciones de seguridad y medidas de autenticacion del propio mongodb

por ejemplo: con nuestra cadena de conexion,, podemos estar ingresando un password erroneo, o un usuarios equivocado. las contraseñas son sensibles a las mayusculas y minusculas
- si el usuario no existe
podemos crear  un nuevo usuario de base d e datos. y generar las credenciales para acceder desde mongosh. 

dentro de la consola en la barra izq. en la seccion **security** encontramos nuestros accesos a la base de datos. 

si tarda mucho en conectar, posiblemente sea porque nuestra ip no alcanza la direccon de nuestra instancia atlas. asi que podemos ingresar a la consola UI de Atlas para verificar esta configuracicon,

en acceso a la red, podemos agregar una nueva direccion ip para que nuestro pc pueda entrar al cluster, es como un grupo de seguridad de una instancia. 

- los clusters de Atlas pueden saturarse de conexiones y alcanzar los limites maximos. tienen un maximo de 500 conexiones al mismo tiempo.

debes propbar estos casos en escenarios de desarrollo, podemos resptablecer la aplicaciones. o subir el tier de neustro cluster. 


