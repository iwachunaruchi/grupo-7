# Página de ejemplo: Usar servicios de Azure en la nube

Este proyecto contiene una página HTML estática que explica servicios de Azure y un `Dockerfile` para construir una imagen que sirve la página con `nginx`.

## Archivos
- [index.html](index.html) - Página principal.
- [styles.css](styles.css) - Estilos.
- [Dockerfile](Dockerfile) - Imagen basada en nginx.

## Construir y ejecutar localmente
```bash
# Construir la imagen
docker build -t azure-web-ejemplo:1.0 .

# Ejecutar localmente
docker run -p 8080:80 azure-web-ejemplo:1.0
# Luego abrir http://localhost:8080
```

## Publicar en Azure (resumen)
Requisitos: tener instalada la Azure CLI (`az`) y estar autenticado con `az login`.

1) Crear un recurso ACR (Azure Container Registry) y subir la imagen:
```bash
az group create --name MyResourceGroup --location eastus
az acr create --resource-group MyResourceGroup --name MyRegistryName --sku Basic
az acr login --name MyRegistryName

docker tag azure-web-ejemplo:1.0 MyRegistryName.azurecr.io/azure-web-ejemplo:1.0
docker push MyRegistryName.azurecr.io/azure-web-ejemplo:1.0
```

2) Desplegar en App Service (Linux container):
```bash
az appservice plan create --name MyPlan --resource-group MyResourceGroup --is-linux --sku B1
az webapp create --resource-group MyResourceGroup --plan MyPlan --name MyWebAppName --deployment-container-image-name MyRegistryName.azurecr.io/azure-web-ejemplo:1.0
```

3) Desplegar en Azure Container Instances (para pruebas rápidas):
```bash
az container create --resource-group MyResourceGroup --name mycontainer --image MyRegistryName.azurecr.io/azure-web-ejemplo:1.0 --dns-name-label mycontainer-uniquelabel --ports 80
```

## Notas rápidas
- Para escenarios de producción con microservicios, considera AKS (Azure Kubernetes Service).
- Usa Managed Identities y Service Principals para CI/CD seguro cuando subas imágenes a ACR.

## Recursos
- https://docs.microsoft.com/azure/
- https://docs.microsoft.com/cli/azure/

## Subir imágenes de prueba a Azure Blob Storage
Se ha añadido una página de prueba `upload.html` que permite subir una imagen directamente desde el navegador.

Pasos rápidos:

1. Genera una SAS de contenedor (ejemplo con Azure CLI) — esto crea una URL que incluye el token SAS:

```bash
# Ejemplo: generar SAS para un contenedor por 1 hora
az storage container generate-sas --account-name <cuenta> --name <contenedor> --permissions acdlrw --expiry "$(date -u -d '1 hour' '+%Y-%m-%dT%H:%MZ')" --output tsv
```

2. Copia la URL del contenedor con SAS (debe verse como `https://<cuenta>.blob.core.windows.net/<contenedor>?sv=...`).
3. Abre `upload.html`, pega la URL del contenedor con SAS, selecciona una imagen y pulsa `Subir`.

Notas de seguridad: usa SAS temporales y permisos mínimos para pruebas. No incrustes SAS en código público.


