## Instalación de JAVA

```bash
# Instalar SDKMAN!
curl -s https://get.sdkman.io | bash

# Cargar SDKMAN! en la sesión actual de la terminal
source ~/.bashrc

# Instalar la distribución Temurin de Java 17 usando SDKMAN!
sdk install java 17.0.10-tem

# (Opcional) Establecer esta versión de Java como predeterminada
sdk default java 17.0.10-tem
```

## Instalación de nextflow

```bash
# Usar root

# Descargar Nextflow
wget -qO- https://get.nextflow.io | bash

# Dar permisos de ejecución al archivo nextflow
chmod 775 nextflow

# Copiar nextflow a /usr/local/bin para que esté disponible globalmente (requiere sudo)
sudo cp nextflow /usr/local/bin/

# Ejecutar Nextflow para comprobar la instalación
nextflow
```

## Instalación de singularity

```bash
# Usar root

# Agregar el repositorio de backports de Go
sudo add-apt-repository ppa:longsleep/golang-backports

# Actualizar la lista de paquetes
sudo apt-get update

# Instalar dependencias necesarias (autoconf, automake, Go, Git, librerías, etc.)
sudo apt-get install -y \
    autoconf \
    automake \
    cryptsetup \
    fuse2fs \
    git \
    fuse \
    libfuse-dev \
    libglib2.0-dev \
    libseccomp-dev \
    libtool \
    pkg-config \
    runc \
    squashfs-tools \
    squashfs-tools-ng \
    uidmap \
    wget \
    zlib1g-dev \
    golang-go

# Verificar la versión de Go instalada
go version

# Clonar el repositorio de Singularity con submódulos
git clone --recurse-submodules https://github.com/sylabs/singularity.git

# Cambiar al directorio del repositorio
cd singularity

# Configurar la compilación (sin libsubid)
./mconfig --without-libsubid

# Compilar el proyecto en el directorio builddir
make -C builddir

# Instalar Singularity (requiere permisos de superusuario)
sudo make -C builddir install

# Verificar la versión instalada de Singularity
singularity --version
```

## Instalación de docker

```bash
# Usar root

# Instalar paquetes necesarios para usar repositorios HTTPS
sudo apt install apt-transport-https ca-certificates curl gnupg lsb-release

# Crear el directorio para llaves GPG de Docker
sudo mkdir -p /etc/apt/keyrings

# Descargar y guardar la llave GPG de Docker
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Agregar el repositorio oficial de Docker
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Actualizar el índice de paquetes
sudo apt update

# Instalar Docker Engine, CLI y plugins
sudo apt install docker-ce docker-ce-cli containerd.io docker-compose-plugin

# Probar la instalación de Docker
docker run hello-world

# (Opcional) Crear el grupo docker si no existe
sudo groupadd docker

# (Opcional) Añadir tu usuario al grupo docker (reemplaza 'fguzman' por tu usuario si es diferente)
sudo usermod -aG docker user
```

## Instalación de NVIDIA Container Toolkit 

```bash
# Usar root

# Actualizar el índice de paquetes
sudo apt-get update

# Instalar NVIDIA Container Toolkit
sudo apt-get install -y nvidia-container-toolkit

# Reiniciar el servicio de Docker para aplicar los cambios
sudo systemctl restart docker
```

## Basecalling con seqtagger 

```bash
# Crear el directorio de trabajo y entrar en él
mkdir -p seqtagger
cd seqtagger

# Descargar los datos de prueba de seqtagger (ignorar index.html)
wget https://public-docs.crg.es/enovoa/public/seqtagger/test_data/ \
    -q --show-progress -r -c -nc -np -nH --cut-dirs=3 --reject="index.html*"

# Crear un alias para usar seqtagger en Docker con soporte GPU y montaje de volumen
alias seqtagger="docker run --gpus all -u $UID:$GID -v `pwd`:/data lpryszcz/seqtagger"

# Demultiplexar datos de mRNA usando seqtagger
seqtagger mRNA -k models/b04_RNA004 -r -i /data/test_data/RNA004 -o /data/demux

# Separar archivos fast5 por código de barras usando el script de seqtagger
seqtagger fast5_split_by_barcode.py -b 50 -i /data/demux/RNA004.demux.tsv.gz -f /data/test_data/RNA004 -o /data/demux/RNA004
```

## Basecalling con master of pores 

```bash
# Clonar el repositorio master_of_pores con submódulos, solo la última versión (shallow clone)
git clone --depth 1 --recurse-submodules https://github.com/biocorecrg/master_of_pores.git

# Entrar al directorio del repositorio
cd master_of_pores

# Ejecutar el pipeline de preprocesamiento con Nextflow, usando Singularity y archivo de parámetros personalizado
nextflow run mop_preprocess.nf -with-singularity -params-file params.yaml -profile local
```
