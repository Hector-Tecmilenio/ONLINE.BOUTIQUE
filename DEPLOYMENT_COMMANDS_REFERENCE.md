# 🚀 Online Boutique + HashiCorp Vault - Referencia de Comandos de Despliegue

## 📋 Configuración de Prerrequisitos

### Configuración Inicial del Entorno (WSL Ubuntu)
```bash
# Configurar entorno Python si es necesario
configure_python_environment

# Instalar/Actualizar kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
export PATH="/usr/local/bin:$PATH"
echo 'export PATH="/usr/local/bin:$PATH"' >> ~/.bashrc

# Configurar AWS CLI (si no se ha hecho ya)
aws configure
```

---

## 🏗️ **INICIANDO TODO** (Infraestructura Completa)

### Paso 1: Desplegar Infraestructura con Terraform
```bash
# Navegar al directorio terraform (usar sistema de archivos Linux nativo para mejor rendimiento)
cp -r "/mnt/c/online-boutique-devops-microservices-project-oldfiles - Copy/terraform" ~/terraform-vault
cd ~/terraform-vault

# Inicializar y desplegar infraestructura
terraform init
terraform plan
terraform apply

# Obtener información del cluster
terraform output
```

### Paso 2: Configurar kubectl para EKS
```bash
# Actualizar kubeconfig para conectar al cluster EKS
aws eks update-kubeconfig --region us-east-2 --name online-boutique-cluster

# Verificar conexión
kubectl get nodes
```

### Paso 3: Desplegar Aplicación Online Boutique
```bash
# Navegar a la raíz del proyecto
cd "/mnt/c/online-boutique-devops-microservices-project-oldfiles - Copy"

# Desplegar aplicación usando kustomize
kubectl apply -k kustomize/

# Asegurar que el servicio LoadBalancer se crea
kubectl apply -f kustomize/base/frontend.yaml

# Esperar a que los pods estén listos (puede tomar 2-5 minutos)
kubectl get pods -w
# Presionar Ctrl+C cuando los pods estén ejecutándose
```

### Paso 4: Obtener URLs de Aplicación
```bash
# Obtener URL de Online Boutique
kubectl get service frontend-external

# Obtener información de Vault (desde salida de terraform)
cd ~/terraform-vault
terraform output vault_ui_url
terraform output vault_server_ip
```

### Paso 5: Inicializar HashiCorp Vault (Solo Primera Vez)
```bash
# SSH al servidor Vault
ssh -i ~/.ssh/vault-server-key ubuntu@$(terraform output -raw vault_server_ip)

# Ejecutar el script de configuración automatizado
chmod +x /tmp/vault-setup.sh && /tmp/vault-setup.sh

# Guardar las claves de unseal y el token root desde:
cat vault-init-output.txt

# Salir de la sesión SSH
exit
```

---

## ✅ **VERIFICANDO ESTADO** (Todo Ejecutándose)

### Verificar Estado de Infraestructura
```bash
# Verificar nodos del cluster EKS
kubectl get nodes

# Verificar todos los pods ejecutándose
kubectl get pods --all-namespaces

# Verificar servicios e IPs externas
kubectl get services --all-namespaces
```

### Verificar URLs de Aplicación
```bash
# Aplicación Online Boutique
kubectl get service frontend-external

# HashiCorp Vault
cd ~/terraform-vault
terraform output vault_ui_url
```

### Verificar Estado de Vault
```bash
# SSH al servidor Vault
ssh -i ~/.ssh/vault-server-key ubuntu@$(cd ~/terraform-vault && terraform output -raw vault_server_ip)

# Verificar estado de Vault
export VAULT_ADDR='http://localhost:8200'
vault status

# Salir de la sesión SSH
exit
```

---

## 🛑 **PARANDO TODO** (Apagado Seguro)

### Opción 1: Parar Solo la Aplicación (Mantener Infraestructura)
```bash
# Navegar a la raíz del proyecto
cd "/mnt/c/online-boutique-devops-microservices-project-oldfiles - Copy"

# Eliminar la aplicación de Kubernetes
kubectl delete -k kustomize/

# O eliminar todo en el namespace por defecto
kubectl delete all --all -n default
```

### Opción 2: Destruir Todo (Desmantelamiento Completo)
```bash
# Paso 1: Eliminar aplicación de Kubernetes primero
cd "/mnt/c/online-boutique-devops-microservices-project-oldfiles - Copy"
kubectl delete -k kustomize/

# Paso 2: Esperar limpieza de LoadBalancer (¡IMPORTANTE!)
echo "⏳ Esperando 60 segundos para limpieza de LoadBalancer de AWS..."
sleep 60

# Paso 3: Destruir toda la infraestructura
cd ~/terraform-vault
terraform destroy
# Escribir 'yes' cuando se solicite

# Paso 4: Limpiar copia local de terraform (opcional)
rm -rf ~/terraform-vault
```

---

## 🔐 **COMANDOS ESPECÍFICOS DE VAULT**

### Acceder a la UI de Vault
```bash
# Obtener URL de Vault
cd ~/terraform-vault
terraform output vault_ui_url
# Abrir esta URL en tu navegador e iniciar sesión con el token root
```

### Comandos CLI de Vault
```bash
# SSH al servidor Vault
ssh -i ~/.ssh/vault-server-key ubuntu@$(cd ~/terraform-vault && terraform output -raw vault_server_ip)

# Establecer dirección de Vault
export VAULT_ADDR='http://localhost:8200'

# Iniciar sesión con token root
vault login
# Ingresar tu token root cuando se solicite

# Operaciones básicas de Vault
vault status
vault secrets list
vault auth list

# Ejemplo: Almacenar un secreto
vault kv put secret/myapp username=admin password=secret123

# Ejemplo: Recuperar un secreto
vault kv get secret/myapp

# Salir de la sesión SSH
exit
```

---

## 🚨 **NOTAS IMPORTANTES**

### Advertencias de Destrucción de Terraform
- ⚠️ **SIEMPRE** eliminar servicios LoadBalancer de Kubernetes antes de ejecutar `terraform destroy`
- ⚠️ Los LoadBalancers crean recursos AWS que Terraform no rastrea
- ⚠️ Esperar al menos 60 segundos después de eliminar servicios de Kubernetes antes de destruir infraestructura
- ⚠️ Se te pedirá confirmar la destrucción - escribir `yes`

### Seguridad de Vault
- 🔐 **GUARDAR** las claves de unseal de Vault y el token root de forma segura
- 🔐 Necesitas 3 de 5 claves de unseal para desbloquear Vault si se sella
- 🔐 El token root proporciona acceso administrativo completo
- 🔐 Los datos de Vault se almacenan en la instancia EC2 en `/opt/vault/data`

### Optimización de Costos
- 💰 Recordar destruir recursos cuando no se necesiten para evitar cargos de AWS
- 💰 El cluster EKS, instancias EC2 y LoadBalancers incurren en costos continuos
- 💰 Considerar parar (no destruir) para apagados temporales

---

## 🔗 **URLs de Acceso Rápido** (Después del Despliegue)

Una vez que todo esté ejecutándose, tendrás:

1. **Aplicación Online Boutique**: 
   - Obtener URL: `kubectl get service frontend-external`
   - Ejemplo: `http://aedf8e8410ec84785b3a7a1aff2a08c7-17630928.us-east-2.elb.amazonaws.com`

2. **UI de HashiCorp Vault**: 
   - Obtener URL: `cd ~/terraform-vault && terraform output vault_ui_url`
   - Ejemplo: `http://3.147.124.108:8200`

3. **Acceso SSH a Vault**: 
   - Comando: `ssh -i ~/.ssh/vault-server-key ubuntu@<vault-server-ip>`

---

## 📞 **Comandos Rápidos de Solución de Problemas**

```bash
# Verificar logs de pods si algo no funciona
kubectl logs <pod-name>

# Describir un pod problemático
kubectl describe pod <pod-name>

# Verificar eventos
kubectl get events --sort-by=.metadata.creationTimestamp

# Reiniciar un despliegue
kubectl rollout restart deployment/<deployment-name>

# Verificar estado de terraform
cd ~/terraform-vault
terraform state list
terraform show
```

---

**📝 ¡Mantén esta referencia a mano para gestionar tu infraestructura completa!**