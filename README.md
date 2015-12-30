<h1 align="center"><font color="#DF0101"><b>INSTITUTO POLITÉCNICO NACIONAL</b></font></h1>
<h2 align="center"><font color="#00FF00">U.P.I.I.C.S.A.</font></h2>
<br><br/>
<h3 align="center"><b>ALMACENAMIENTO DE DATOS Y SU ADMINISTRACIÓN</b></h3>
<h3 align="center">TAREA 6: ELEMENTOS INTERMEDIOS DE R</h3>
<br><br/>

#### **NOMBRE: JUAN JOSÉ LAIR MARTÍNEZ**
#### **BOLETA: B150848**

<br><br/>
<h4 align="left"><font color="##FF0000"><b>OBJETIVO</b></font></h4>
Familiarizar al alumno con las acciones de automatización de adquisición de datos en R como una base metódica en la búsqueda de reproducibilidad de resultados y replicabilidad de acciones relacionadas con el análisis de datos.
<br><br/>
<h4 align="left"><font color="##FF0000"><b>ACTIVIDADES</b></font></h4>

1.- Revise el sitio de los National Centers for Environmental Information (antes National Climatic Data Center, NCDC) de la National Oceanic and Atmospheric Administration (NOAA) y localice la liga de acceso a conjuntos de datos (Data Access).

2.- Localice la categoría de datos de clima severo (Severe Weather) y, de ésta, la liga a la página de la base de datos de eventos de tormentas (Storm Events Database).

3.- Localice la liga de descarga de datos a granel (bulk).

4.- Via HTTP revise la documentación y descripción de archivos y datos disponibles.

5.- Anote los nombre de los archivos relacionados con decesos (fatalities) para un rango de una década con los que desea trabajar.

6.- Cree un script R que haga lo siguiente:

```{r}
# Se eliminan las variables en caso de existir

if( exists("PACKAGES")) {
  rm(PACKAGES)
}
if( exists("package")) {
  rm(package)
}
if( exists("WORKDIR")) {
  rm(WORKDIR)
}
if( exists("URLDOWNLOAD")) {
  rm(URLDOWNLOAD)
}
if( exists("FILES")) {
  rm(FILES)
}
if( exists("FILESDOWNLOAD")) {
  rm(FILESDOWNLOAD)
}
if( exists("file")) {
  rm(file)
}
if( exists("files")) {
  rm(files)
}
if( exists("Fatalities") ){
    rm(Fatalities)
}
if( exists("data") ){
    rm(data)
}
```

a) Cargar los paquetes necesarios si éstos no han sido cargados.

```{r}
PACKAGES<-list("R.oo", "R.utils")
for (package in PACKAGES ) {
  if (!require(package, character.only=T, quietly=T)) {
      install.packages(package)
      library(package)
  }
}
```

b) Establezca el directorio de trabajo.
c) Valide la existencia y cree un directorio de descarga, de no existir.
d) Valide la existencia de un directorio para los conjuntos de datos y lo cree de ser necesario.

```{r}
# Se establece el directorio de trabajo, si no existe, se crea

if( dir.exists("c:/AdmonDatos/Tarea6/"))
{
  print("El directorio ya existe")
} else {
  dir.create(file.path("c:/AdmonDatos/Tarea6/"), recursive=TRUE) 
}

# Se establece el directorio de trabajo y se verifica

WORKDIR<-"c:/AdmonDatos/Tarea6/"
setwd(WORKDIR)
getwd()
```

e) El script contará con la lista de los nombres de archivos a trabajar. Si el archivo no está presente en el directorio de datos deberá buscarse en el directorio de descarga, si no está presente deberá descargarse.
f) Una vez descargado el archivo, y ya presente en el sistema deberá descompactarse dejando el archivo de datos en el directorio apropiado.

```{r}
# Se crean las variables para descargar los archivos del sitio

URLDOWNLOAD<-"http://www1.ncdc.noaa.gov/pub/data/swdi/stormevents/csvfiles/"

# FILES es la lista de archivos para ser descargados.
# En caso de que no existan los archivos, se descargan
# FILES es la lista de archivos descargados y descomprimidos

FILES<-list("StormEvents_details-ftp_v1.0_d1971_c20150826.csv","StormEvents_details-ftp_v1.0_d1972_c20150826.csv","StormEvents_details-ftp_v1.0_d1973_c20150826.csv","StormEvents_details-ftp_v1.0_d1974_c20150826.csv","StormEvents_details-ftp_v1.0_d1975_c20150826.csv","StormEvents_details-ftp_v1.0_d1976_c20150826.csv","StormEvents_details-ftp_v1.0_d1977_c20150826.csv","StormEvents_details-ftp_v1.0_d1978_c20150826.csv","StormEvents_details-ftp_v1.0_d1979_c20150826.csv","StormEvents_details-ftp_v1.0_d1980_c20150826.csv")

for( file in FILES ){
  # Se valida si el archivo descompactado ya existe en el área de datos.
  if( ! file.exists( WORKDIR )) {
    # Si no existe se busca el archivo compactado en el área de descarga.
    if( ! file.exists( WORKDIR ) ){
        downloadFile( paste(URLDOWNLOAD, paste(file, "gz", sep="."), sep = "/"), file, skip=TRUE, overwrite=TRUE ) 
    }
    if( file.exists( WORKDIR )){
        print(paste("Se descomprime el archivo", paste(file, "gz", sep="."), sep=": "))
        gunzip( file, overwrite=TRUE, remove=TRUE )
    }
  }
}
```

g) Una vez con todos los archivos presentes, se leerán todos los archivos, mostrando por cada uno el número de registros y estos archivos se fusionarán en una sola estructura de datos. Para esto último es necesario considerar lo siguiente:

    - Si la intención es que este script pueda ser ejecutado varias veces, es importante que la estructura de datos sea limpiada en cada ejecución.
    
    - La lectura de los archivos debe considerar dos casos: La lectura inicial, en la que la estructura es inicialmente llenada con la lectura del primer archivo:
    
    - y las lectura subsecuentes (la unión de datos se logra empleando una variable temporal y la función rbind().):
    
```{r}
# Se muestra el número de registros por cada archivo; así como la recopilación de todos
# mostrando el total de registros por cada archivo y el total de todos ellos

for( files in FILES ){
    if( !exists("Fatalities" ) ) {
        Fatalities<-read.csv( files, header=T, sep=",", na.strings="")
        print(paste("Registros del archivo", paste(files, nrow(Fatalities), sep=": "), sep=" "))
    } else {
        data<-read.csv(files, header=T, sep=",", na.strings="")
        print(paste("Registros del archivo", paste(files, nrow(data), sep=": "), sep=" "))
        Fatalities<-rbind(Fatalities,data)
    }
}
```

h) Al final se desplegará el número de registros del data frame creado, que deberá ser la suma de los registros de cada archivo

```{r}
print(paste("Total de registros: ", nrow(Fatalities), sep = " "))
```
<br><br/>

**Nota: Se recalca que este script deberá poder ser ejecutado varias veces, evitando cargar las librerías nuevamente y descargar los archivos que ya se encuentren en el sistema.**