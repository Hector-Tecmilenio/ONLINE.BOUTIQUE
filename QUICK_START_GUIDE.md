# 🚀 **GUÍA DE INICIO/PARADA RÁPIDA** - Orden Paso a Paso

## 🏗️ **CONFIGURACIÓN INICIAL** (Creación de Infraestructura)

### Paso 1: Configurar Entorno
```bash
# Start WSL Ubuntu terminal
# Navigate to terraform directory
cp -r "/mnt/c/online-boutique-devops-microservices-project-oldfiles - Copy/terraform" ~/terraform-vault
cd ~/terraform-vault
```

### Paso 2: Desplegar Infraestructura con Terraform
```bash
terraform init
terraform plan
terraform apply
# Escribir 'yes' cuando se solicite
```

### Paso 3: Inicializar Vault (Solo Primera Vez)
```bash
# Copiar y ejecutar el script de despliegue
cp "/mnt/c/online-boutique-devops-microservices-project-oldfiles - Copy/ansible/deploy-vault.sh" ~/ansible-vault-project/
cd ~/ansible-vault-project
./deploy-vault.sh

# SSH a Vault y guardar credenciales
ssh -i ~/.ssh/vault-server-key ubuntu@$(cd ~/terraform-vault && terraform output -raw vault_server_ip)
cat vault-init-output.txt  # ¡GUARDAR ESTAS CREDENCIALES!
exit
```

---

## 🔄 **USO DIARIO** (Iniciar/Parar Aplicaciones)

### 🚀 **PARA INICIAR TODO:**

#### Paso 1: Configurar kubectl
```bash
# Se puede ejecutar desde cualquier directorio
aws eks update-kubeconfig --region us-east-2 --name online-boutique-cluster
```

#### Paso 2: Navegar al Proyecto
```bash
cd "/mnt/c/online-boutique-devops-microservices-project-oldfiles - Copy"
```

#### Paso 3: Iniciar Aplicación
```bash
kubectl apply -k kustomize/
```

#### Paso 4: Obtener URLs
```bash
# URL de Online Boutique
kubectl get service frontend-external

# URL de Vault (si se necesita)  
cd ~/terraform-vault
terraform output vault_ui_url
```

### 🛑 **PARA PARAR SOLO LA APLICACIÓN** (Mantener Infraestructura):
```bash
cd "/mnt/c/online-boutique-devops-microservices-project-oldfiles - Copy"
kubectl delete -k kustomize/
```

---

## 💥 **DESMANTELAMIENTO COMPLETO** (Parar Facturación AWS)

### ⚠️ **Paso 1: Eliminar Aplicación Primero** (¡ORDEN CRÍTICO!)
```bash
cd "/mnt/c/online-boutique-devops-microservices-project-oldfiles - Copy"
kubectl delete -k kustomize/

# Verificar que el LoadBalancer se haya ido
kubectl get services
```

### ⏳ **Paso 2: Esperar Limpieza de AWS** (¡IMPORTANTE!)
```bash
echo "⏳ Esperando 60 segundos para limpieza de LoadBalancer de AWS..."
sleep 60
```

### 🔥 **Paso 3: Destruir Infraestructura**
```bash
cd ~/terraform-vault
terraform destroy
# Escribir 'yes' cuando se solicite

# Limpiar copia local
rm -rf ~/terraform-vault
```

---

## 🎯 **ACCESO RÁPIDO** (Cuando Todo Está Ejecutándose)

### URLs de Aplicación:
- **Online Boutique**: `kubectl get service frontend-external`
- **Vault UI**: http://3.147.124.108:8200 (or `terraform output vault_ui_url`)

### Verificaciones de Estado:
```bash
# Verificar si la app está ejecutándose
kubectl get pods

# Verificar servicios
kubectl get services

# Verificar estado de Vault
ssh -i ~/.ssh/vault-server-key ubuntu@3.147.124.108
export VAULT_ADDR='http://localhost:8200'
vault status
exit
```

---

## 💡 **RECORDAR:**

- **Infraestructura** (Terraform) = Construir una vez, cuesta dinero mientras está ejecutándose
- **Aplicación** (Kubernetes) = Se puede iniciar/parar rápidamente, sin costo extra
- **Vault** = Siempre ejecutándose cuando la infraestructura existe
- **Para proyectos escolares**: ¡Siempre ejecutar desmantelamiento completo cuando termine!

## ⚠️ **ORDEN CRÍTICO DE DESMANTELAMIENTO:**
1. Eliminar aplicación Kubernetes PRIMERO
2. Esperar 60 segundos
3. Luego destruir infraestructura Terraform
4. ¡Esto previene recursos AWS huérfanos y facturación inesperada!