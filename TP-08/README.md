# Trabajo Práctico Número 7

## Prerequisitos

![nada](images/1.1.png)
![nada](images/1.2.png)

## Punto 1: Crear archivos DockerFile para nuestros proyectos de Back y Front

![nada](images/1.3.png)
![nada](images/1.4.png)

## Punto 2: Crear un recurso ACR en Azure Portal siguiendo el instructivo 5.1

![nada](images/1.5.png)
![nada](images/1.6.png)
![nada](images/1.7.png)

## Punto 3: Modificar nuestro pipeline en la etapa de Build y Test

```
    # Publicar Dockerfile de Back
    - task: PublishPipelineArtifact@1
      displayName: 'Publicar Dockerfile de Back'
      inputs:
        targetPath: '$(Build.SourcesDirectory)/docker/api/Dockerfile'
        artifact: 'dockerfile-back'
```

```
    # Publicar Dockerfile de Front
    - task: PublishPipelineArtifact@1
      displayName: 'Publicar Dockerfile de Front'
      inputs:
        targetPath: '$(Build.SourcesDirectory)/docker/front/Dockerfile'
        artifact: 'dockerfile-front'
```

![nada](images/1.8.png)

## Punto 4: En caso de no contar en nuestro proyecto con una ServiceConnection a Azure Portal para el manejo de recursos, agregar una service connection a Azure Resource Manager como se indica en instructivo 5.2

![nada](images/1.9.png)
![nada](images/1.10.png)
![nada](images/1.11.png)
![nada](images/1.12.png)

## Punto 5: Agregar a nuestro pipeline variables

![nada](images/1.13.png)

## Punto 6: Agregar a nuestro pipeline una nueva etapa que dependa de nuestra etapa de Build y Test

```
# #----------------------------------------------------------
# ### STAGE BUILD DOCKER IMAGES Y PUSH A AZURE CONTAINER REGISTRY
# #----------------------------------------------------------

- stage: DockerBuildAndPush
  displayName: 'Construir y Subir Imágenes Docker a ACR'
  dependsOn: BuildAndTest
  jobs:
   - job: docker_build_and_push
     displayName: 'Construir y Subir Imágenes Docker a ACR'
     pool:
       vmImage: 'ubuntu-latest'

     steps:
       - checkout: self

       #----------------------------------------------------------
       # BUILD DOCKER BACK IMAGE Y PUSH A AZURE CONTAINER REGISTRY
       #----------------------------------------------------------

       - task: DownloadPipelineArtifact@2
         displayName: 'Descargar Artefactos de Back'
         inputs:
           buildType: 'current'
           artifactName: 'api-drop'
           targetPath: '$(Pipeline.Workspace)/drop-back'

       - task: DownloadPipelineArtifact@2
         displayName: 'Descargar Dockerfile de Back'
         inputs:
           buildType: 'current'
           artifactName: 'dockerfile-back'
           targetPath: '$(Pipeline.Workspace)/dockerfile-back'

       - task: AzureCLI@2
         displayName: 'Iniciar Sesión en Azure Container Registry (ACR)'
         inputs:
           azureSubscription: '$(ConnectedServiceName)'
           scriptType: bash
           scriptLocation: inlineScript
           inlineScript: |
             az acr login --name $(acrLoginServer)

       - task: Docker@2
         displayName: 'Construir Imagen Docker para Back'
         inputs:
           command: build
           repository: $(acrLoginServer)/$(backImageName)
           dockerfile: $(Pipeline.Workspace)/dockerfile-back/Dockerfile
           buildContext: $(Pipeline.Workspace)/drop-back
           tags: 'latest'

       - task: Docker@2
         displayName: 'Subir Imagen Docker de Back a ACR'
         inputs:
           command: push
           repository: $(acrLoginServer)/$(backImageName)
           tags: 'latest'
```

## Punto 7: Ejecutar el pipeline y en Azure Portal acceder a la opción Repositorios de nuestro recurso Azure Container Registry. Verificar que exista una imagen con el nombre especificado en la variable backImageName asignada en nuestro pipeline

![nada](images/1.14.png)
![nada](images/1.15.png)

## Punto 8: Agregar tareas para generar imagen Docker de Front (DESAFIO)

```
       #----------------------------------------------------------
       # BUILD DOCKER FRONT IMAGE Y PUSH A AZURE CONTAINER REGISTRY
       #----------------------------------------------------------

       - task: DownloadPipelineArtifact@2
         displayName: 'Descargar Artefactos de Front'
         inputs:
           buildType: 'current'
           artifactName: 'front-drop'
           targetPath: '$(Pipeline.Workspace)/drop-front'

       - task: DownloadPipelineArtifact@2
         displayName: 'Descargar Dockerfile de Front'
         inputs:
           buildType: 'current'
           artifactName: 'dockerfile-front'
           targetPath: '$(Pipeline.Workspace)/dockerfile-front'

       - task: AzureCLI@2
         displayName: 'Iniciar Sesión en Azure Container Registry (ACR)'
         inputs:
           azureSubscription: '$(ConnectedServiceName)'
           scriptType: bash
           scriptLocation: inlineScript
           inlineScript: |
             az acr login --name $(acrLoginServer)

       - task: Docker@2
         displayName: 'Construir Imagen Docker para Front'
         inputs:
           command: build
           repository: $(acrLoginServer)/$(frontImageName)
           dockerfile: $(Pipeline.Workspace)/dockerfile-front/Dockerfile
           buildContext: $(Pipeline.Workspace)/drop-front/employee-crud-angular/browser
           tags: 'latest'

       - task: Docker@2
         displayName: 'Subir Imagen Docker de Front a ACR'
         inputs:
           command: push
           repository: $(acrLoginServer)/$(frontImageName)
           tags: 'latest'
```

![nada](images/1.16.png)
![nada](images/1.17.png)

## Punto 9: Agregar a nuestro pipeline una nueva etapa que dependa de nuestra etapa de Construcción de Imagenes Docker y subida a ACR

![nada](images/1.18.png)
![nada](images/1.19.png)
![nada](images/1.20.png)
![nada](images/1.21.png)

## Punto 10: Ejecutar el pipeline y en Azure Portal acceder al recurso de Azure Container Instances creado. Copiar la url del contenedor y navegarlo desde browser. Verificar que traiga datos.

![nada](images/1.22.png)
![nada](images/1.23.png)
![nada](images/1.24.png)

## Punto 11: Agregar tareas para generar un recurso Azure Container Instances que levante un contenedor con nuestra imagen de front (DESAFIO)

![nada](images/1.25.png)
![nada](images/1.26.png)
![nada](images/1.27.png)
![nada](images/1.28.png)
![nada](images/1.29.png)
![nada](images/1.30.png)
![nada](images/1.31.png)
![nada](images/1.32.png)
![nada](images/1.33.png)

## Punto 12: Agregar tareas para correr pruebas de integración en el entorno de QA de Back y Front creado en ACI.

![nada](images/1.34.png)
![nada](images/1.35.png)
![nada](images/1.36.png)
![nada](images/1.37.png)

## DESAFIO:

### Agregar tareas para generar imagen Docker de Front. (Realizado en el Punto 4.1.8)

### Agregar tareas para generar en Azure Container Instances un contenedor de imagen Docker de Front. (Realizado en el Punto 4.1.11)

### Agregar tareas para correr pruebas de integración en el entorno de QA de Back y Front creado en ACI. (Realizado en el Punto 4.1.12)

### Agregar etapa que dependa de la etapa de Deploy en ACI QA y genere contenedores en ACI para entorno de PROD. (DEBAJO)

![nada](images/1.38.png)
![nada](images/1.39.png)
![nada](images/1.40.png)
![nada](images/1.41.png)
![nada](images/1.42.png)
![nada](images/1.43.png)
![nada](images/1.44.png)
![nada](images/1.45.png)
![nada](images/1.46.png)
![nada](images/1.47.png)
![nada](images/1.48.png)
![nada](images/1.49.png)
![nada](images/1.50.png)
![nada](images/1.51.png)
![nada](images/1.52.png)
