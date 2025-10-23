# Online Boutique - Referencia Rápida

## Iniciando la Aplicación

### Paso 1: Crear Infraestructura (si fue destruida)
```powershell
# Navegar al directorio terraform
cd terraform

# Inicializar, planificar y aplicar infraestructura
terraform init
terraform plan
terraform apply
# Escribir 'yes' cuando se solicite
```

### Paso 2: Configurar kubectl
```powershell
# Configurar kubectl (ejecutar una vez por máquina o después de recrear infraestructura)
aws eks update-kubeconfig --region us-east-2 --name online-boutique-cluster
```

### Paso 3: Desplegar Aplicación
```powershell
# Navegar al directorio del proyecto
cd C:\online-boutique-devops-microservices-project

# Desplegar todos los microservicios
kubectl apply -k kustomize/

# Asegurar que el servicio LoadBalancer existe
kubectl apply -f kustomize/base/frontend.yaml

# Obtener URL de la aplicación
kubectl get service frontend-external
```

### Verificar Estado
```powershell
# Verificar todos los pods
kubectl get pods

# Verificar servicios
kubectl get services

# Verificar servicio específico
kubectl get service frontend-external

# Información detallada del pod
kubectl describe pod <pod-name>
```

## Destruyendo Recursos

### Eliminar Solo la Aplicación
```powershell
# Eliminar recursos de Kubernetes
kubectl delete -k kustomize/

# O eliminar todo en el namespace por defecto
kubectl delete all --all -n default
```

### Eliminar Todo (Aplicación + Infraestructura)
```powershell
# 1. Eliminar aplicación de Kubernetes primero
kubectl delete -k kustomize/

# 2. Esperar 2-3 minutos para limpieza de LoadBalancer

# 3. Destruir infraestructura AWS
cd terraform
terraform destroy
# Escribir 'yes' cuando se solicite
```

## Monitoreo y Solución de Problemas

### Ver Logs
```powershell
# Ver logs para servicio específico
kubectl logs deployment/frontend
kubectl logs deployment/cartservice

# Seguir logs en tiempo real
kubectl logs -f deployment/frontend
```

### Escalar Servicios
```powershell
# Escalar un servicio
kubectl scale deployment frontend --replicas=3

# Verificar escalado
kubectl get pods -l app=frontend
```

### Port Forward (para testing local)
```powershell
# Reenviar puerto local al servicio
kubectl port-forward service/frontend 8080:80

# Acceder vía http://localhost:8080
```

## Información de Infraestructura

- **Nombre del Cluster**: online-boutique-cluster
- **Región**: us-east-2
- **Grupo de Nodos**: online-boutique-node-group
- **Registro ECR**: 094121082922.dkr.ecr.us-east-2.amazonaws.com

## Notas Importantes

1. Siempre eliminar servicios LoadBalancer antes de destruir infraestructura Terraform
2. La aplicación usa imágenes de contenedor pre-construidas de Google
3. Los repositorios ECR están disponibles para imágenes personalizadas si se necesitan
4. La URL del LoadBalancer cambia cada vez que se recrea