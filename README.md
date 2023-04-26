# Workshop Functions

### Requerimientos:

- Cuenta de Oracle Cloud Infrastructure(test gratuito https://www.oracle.com/cloud/free/)
- Cuenta de Github (https://github.com/signup?ref_cta=Sign+up&ref_loc=header+logged+out&ref_page=%2F&source=header-home)
- Poseer servidor linux con las siguientes dependencias instaladas:
  - git
  - python36-oci-cli
  - fn 

### ¿Qué vamos a hacer?
Todo el código está en https://github.com/whiplash0104/Workshop-Function


### Paso a Paso

1. Crear Cuenta en git, usuar correo empresarial o personal (github.com)


https://user-images.githubusercontent.com/14284928/219779574-ad656998-a63d-402a-87cd-b9658c84e401.mov


2. Creación de compartment con el nombre **FunctionTest**

https://user-images.githubusercontent.com/14284928/228681682-6075d0de-1227-451c-a74a-74c880238f47.mov



3. Creación de VCN con el nombre **VCNFunction** dentro del compartment **FunctionTest** recién creado

https://user-images.githubusercontent.com/14284928/228682080-54806ccb-07a4-4c7d-8ab4-3cb2ca32c705.mov


4. Creación de grupo dinámico con el nombre **FunctionGroup** 
```
Menu principal > Identity & Security > Dynamic Groups > Create Dynamic Group

Name: FunctionGroup
Description: Grupo funciones
Rule 1: ALL {resource.type = 'fnfunc'}
```

https://user-images.githubusercontent.com/14284928/228682912-9d82ec85-b83e-45f2-825b-df364f230566.mov



5. Creación de políticas con el nombre **FunctionPolicies**, estas deben ir en el compartment **root** 

```
Menu principal > Identity & Security > Policies

Name: FunctionPolicies
Description: FunctionGroup Policies
Policy Builder:  Seleciconar Show manual editor y pegar las siguientes reglas

Allow dynamic-group FunctionGroup to manage functions-family in tenancy
Allow dynamic-group FunctionGroup to use virtual-network-family in tenancy
Allow dynamic-group FunctionGroup to manage repos in tenancy
Allow dynamic-group FunctionGroup to inspect object-family in tenancy
Allow dynamic-group FunctionGroup to manage objects in tenancy
Allow dynamic-group FunctionGroup to manage autonomous-database-family in tenancy
Allow dynamic-group FunctionGroup to use ons-topics in tenancy
```

<img width="410" alt="image" src="https://user-images.githubusercontent.com/14284928/229537938-ed102020-f234-4acd-bd55-00617dc08786.png">

https://user-images.githubusercontent.com/14284928/228683128-d2e6f437-473d-4cdc-bf33-c9a12df88eb1.mov



5. Creación de tópico con el nombre **FunctionTopic** dentro del compartment **FunctionTest**. Una vez creado el tópico se debe crear una suscripción, definir en esta el correo de cada uno.
```
Menú Principal > Developer Services > Notifications > Create Topic

Name: FunctionTopic	
```

Dentro del tópico hacer click en Create Subscription
```
Protocol: Email
Email: felipe.basso@oracle.com
```
<img width="593" alt="image" src="https://user-images.githubusercontent.com/14284928/229538763-8e2153f6-a4e4-4e7a-961f-c17c008d75b1.png">

Importante señalar que se debe aceptar la suscripción, esta llegará al correo

https://user-images.githubusercontent.com/14284928/228685015-576493a1-7664-48f5-b986-dfaf0d069ae9.mov


6. Copiar el OCID del topico y almacenarlo en un archivo de texto, se utilizará más adelante
<img width="269" alt="image" src="https://user-images.githubusercontent.com/14284928/229539039-8b5fb2da-3c89-4607-8afe-d36e7487c5b9.png">

https://user-images.githubusercontent.com/14284928/228685250-3844778c-b99e-4b89-8c14-d5f7c91275ea.mov


7. Creación Autonomous Database dentro del compartment **FunctionTest** con los siguientes parámetros:

```
Display name: FunctionADB
Database name: functionadb
Selecionar la opción "Always Free"
Password: ClavE.012356,         La ',' va incluida en la password
```

https://user-images.githubusercontent.com/14284928/228686294-acce7ee2-2daf-4954-8456-9dbee344cabf.mov



8. Una vez creada la base de datos guardar OICD, Password y DBSVC

Estos deberían ser del tipo:
```
DBSVC: functionadb_high
DB_OCID: ocid1.autonomousdatabase.oc1.iad.XXXXXXXXXXXXXXXXXXXXXXXXXXXX
DBUSER: ADMIN
DBPWD:  ClavE.012356,
```

https://user-images.githubusercontent.com/14284928/228686821-3446ac39-8fbe-417f-96b5-863a9b6e4cb1.mov


9. Cración de token para usuario, llamarlo **FunctionToken** y guardarlo en un archivo de texto, este no se volverá a mostrar
```
User Setins > Auth Tokens > Generate Token 
Name: FunctionToken
```
<img width="255" alt="image" src="https://user-images.githubusercontent.com/14284928/229539693-fce81684-e3a3-43c8-86ce-27e838fd96a9.png">


https://user-images.githubusercontent.com/14284928/228687211-79883498-5203-40a1-b352-8158a1844d32.mov


10. Creación de la aplicación (llamarla **apptest**) dentro del compartment **FunctionTest** usar la VCN **VCNFunction** y la subred **pública**
```
Menú Principal > Developer Services > Function > Applications > Create Application
Name: apptest
VCN in Function: VCNFunction
subnets in Function: Public Subnet-VCNFunction
```
<img width="415" alt="image" src="https://user-images.githubusercontent.com/14284928/229540208-557cd702-1bc9-441f-be38-39537a84be88.png">


https://user-images.githubusercontent.com/14284928/228687760-84778199-ce53-4ef1-92a2-b732a30eb0a6.mov


11. Creación de registry con el nombre **apptest** dentro del compartment **FunctionTest**
```
Menú Principal > Developer Services > Containers & Artifacts > Container Registry > Create Repository
Compartment: FunctionTest
Repository name: apptest
Access: Private
```

https://user-images.githubusercontent.com/14284928/228694517-62f99cfa-4db6-46c5-bc65-45c0c2e0f257.mov


12. Configurar fn cli, según lo indicado al momento de la creación de la función
```
selecionar "Cloud Shell setup"
Click en Launch Cloud Shell y esperar unos segundos.
Si es primera vez que se abre la consola esta nos preguntará si deseamos ver la docuemntación, presionar la letra "N" y dar "Enter"
```

Copiar los comandos que nos entrega la página (estos serán distintos en cada caso)
```
Configurar la región dentro del context de fn (XX-XXXXXX-X Es la región a la que pertenece el tenant)
	fn list context
	fn use context XX-XXXXXX-X
Configurar compartment dentro del context de fn (ocid1.compartment.oc1.XXX Es el compartmente)
	fn update context oracle.compartment-id ocid1.compartment.oc1.XXX
Configurar el registry (NAMESPACE Es el namespace donde fue creado el registry)
	fn update context registry iad.ocir.io/NAMESPACE_PERSONAL/apptest

Login dentro del repositorio
	docker login  -u 'NAMESPACE/USUARIO' -p 'TOKEN'  REGISTRY_URL
	
	Donde:
	NAMESPACE:        Es el namespace donde fue creado el registry
	USUARIO:          Es el usuario de OCI
	TOKEN:            Es el token que se creó en el paso 9
	REGISTRY_URL:     Viene en base a la región https://docs.oracle.com/en-us/iaas/Content/General/Concepts/regions.htm ej: gru.ocir.io

Listar aplicaciones existentes
	fn list apps
```


https://user-images.githubusercontent.com/14284928/234386712-4f51d333-d148-46d2-9a88-2641b8df1053.mov



13. Crear repositorio git workshop-function y clonar repositorio original https://github.com/whiplash0104/Workshop-Function.git
```
Login dentro de github
Click en perfil de usuario -> Your repositories -> New
	Repository Name: workshop-function
```

https://user-images.githubusercontent.com/14284928/234379563-f67be4dd-32b3-4fc9-9b11-6d9f0eda9ddd.mov


14. Modificar los siguientes parámetros del archivo **func.yaml** dentro del directorio Workshop-Function:
<img width="530" alt="image" src="https://user-images.githubusercontent.com/14284928/234380090-547f7258-f352-43f8-85d5-e63c5abde67b.png">
```
  TOPIC_OCID: ocid1.onstopic.oc1.iad.XXXXXXXXXXXXXXXX
  ADB_OCID: ocid1.autonomousdatabase.oc1.iad.XXXXXXXX
  COMPANY: XXXXXXX
  DBPWD: XXXXXXXX
  DBSVC: XXXXXXX_high
```
Estos parámetros fueron almacenados durante los pasos 5, 7 y 8

https://user-images.githubusercontent.com/14284928/234381150-33b58b3b-cda6-492f-91b6-eb4bbdcd5e6f.mov


15. Clonar repositorio git modificado en paso 14 (este link es personal, no se debe clonar el del laboratorio)
```
git clone https://github.com/XXXXXXXXX/workshop-function.git
```

16. Deploy de aplicación, ejecutar comando desde el directorio Workshop-Function
```
cd workshop-function
fn -v deploy --app apptest
```

Esta tarea tarde aproximadamente 4 minutos, denemos esperar el mensaje
```
...
f46abc7ec396: Layer already exists
0.0.125: digest: sha256:6f848be04f6b7db8064c437f8907ebe4e9e446a0f8ebef5437b61dc41a2329e5 size: 1782
Updating function apptest using image iad.ocir.io/id5xe8my93ee/apptest:0.0.125...
Successfully created function: apptest with iad.ocir.io/id5xe8my93ee/apptest:0.0.125
```



https://user-images.githubusercontent.com/14284928/234387625-52aa5198-7f7d-4284-8e00-d34442dfa607.mov



Desde Developer Services > Functions > Applications > apptest (dentro del compartment **FunctionTest**) se podrá ver el detalle de la función. Habilitar el loggin desde la opción Logs 

https://user-images.githubusercontent.com/14284928/228701232-2850b40f-f812-4cb7-aece-601c22c5e2c6.mov


17. Creación de Bucket con el nombre **functionBucket** dentro del compartment **FunctionTest**  
**Se debe habilitar la opción emisión de eventos (Emit Object Events)**




https://user-images.githubusercontent.com/14284928/234388809-7d5f617d-4319-4f11-8941-9527282aecee.mov



18. Configuración de regla para Event Service dentro del compartment **FunctionTest** 
En este punto se crea la regla para que cada vez que se cargue un archivo dentro del bucket se gatillará la ejecución de la aplicación
```
Display Name: FunctionEvent
Condition: Event Type
Service Name: Objetc Storage
Event Type: Object - Create
```
+ Another Condition
```
Condition: attribute
Attribute Name: bucketName
Attribute Values: functionBucket
```

Action
```
Action Type: Functions
Function Compartment: FunctionTest
Function Application: apptest
Function: apptest
```
<img width="737" alt="image" src="https://user-images.githubusercontent.com/14284928/228702179-ac77b704-508e-4f70-8880-5eb928083d6a.png">

```
Rule Logic
MATCH event WHERE (
	eventType EQUALS ANY OF (
		com.oraclecloud.objectstorage.createobject
	)
	AND (
		bucketName MATCHES ANY OF (
			functionBucket
		)
	)
)
```
https://user-images.githubusercontent.com/14284928/228702784-e02325ef-0a3f-457d-b152-e4b16dec94e6.mov

19. Para probar la función
Descargar archivo cvs: https://objectstorage.us-ashburn-1.oraclecloud.com/p/MSmBkReA-TET1pfUpsvX5ZsC6uTFZpU140p7t7uitURUJ9hwOKOI0z0O5mn7stXJ/n/idikzonisftg/b/DataFile/o/Employees.csv

Cargar el archivo csv dentro del bucket creado
Esperar la ejecución de la fn (la primera carga tarda un tiempo aproximado de 40 segundos, posterior a ello la ejecución tarda 3 segundos)
Ir a la base de datos, en Database Action ir al motor SQL y ejecutar la siguiente consulta
```
SELECT * FROM LOAD_TABLE;
```
https://user-images.githubusercontent.com/14284928/229536037-706fe2fa-802f-45ad-83df-10dfb6579601.mov


20. Si la carga se ejecutó de forma correcta además de ver los datos cargados en la DB debería llegar un correo similar al siguiente:
<img width="554" alt="image" src="https://user-images.githubusercontent.com/14284928/229536266-b214dbef-035a-431e-9725-54c8126b8e6e.png">

