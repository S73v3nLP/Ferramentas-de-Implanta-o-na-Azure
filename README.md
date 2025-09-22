## Ferramentas-de-Implanta-o-na-Azure
Lab "Tutorial de Criação de Máquina Virtual utilizando um script em BICEP no AZURE"

# Criação de VM com script BICEP

Claro, Steven! Aqui está o **roteiro completo e detalhado** da etapa de **pré-requisitos** para rodar scripts Bicep no seu ambiente local — ideal para quem está começando com infraestrutura como código no Azure.

---

## 🧰 1. Pré-requisitos para rodar Bicep localmente

### 🎯 Objetivo:
Preparar seu ambiente para criar e implantar recursos no Azure usando **Bicep + Azure CLI**.

---

### 🧩 Etapa 1.1: Instalar o Azure CLI

#### 🔧 Windows:
1. Acesse: [https://aka.ms/installazurecliwindows](https://aka.ms/installazurecliwindows)
2. Baixe o instalador `.msi`
3. Execute e siga o assistente de instalação
  <img width="1062" height="742" alt="image" src="https://github.com/user-attachments/assets/1cb53169-6c14-4973-81f7-a0028420e30b" />


4. Verifique no terminal:
```bash
az version
```
  <img width="992" height="415" alt="image" src="https://github.com/user-attachments/assets/e07d4d80-1234-457f-b572-71a3d6b3f80e" />
  

#### 🔧 Linux (Debian/Ubuntu):
```bash
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```

#### 🔧 macOS:
```bash
brew update && brew install azure-cli
```

---

### 🧩 Etapa 1.2: Instalar o Bicep CLI

> O Bicep CLI é uma extensão do Azure CLI, então você pode instalar direto por ele.

#### Comando universal:
```bash
az bicep install
```

#### Verificar instalação:
```bash
az bicep version
```

---

### 🧩 Etapa 1.3: Fazer login no Azure

```bash
az login
```

- Isso abrirá uma janela no navegador para autenticação.
- Após o login, você verá os detalhes da sua conta e subscriptions no terminal.
  <img width="982" height="612" alt="image" src="https://github.com/user-attachments/assets/030b445d-3ca3-4164-9a2d-12a0a9114025" />

---

### 2. **Criar o arquivo Bicep**
Crie um arquivo chamado `vm-lab.bicep` com o seguinte conteúdo:

```bicep
param vmName string = 'lab-vm'
param location string = resourceGroup().location
param adminUsername string = 'azureuser'
param adminPassword string = 'P@ssword1234!' // Para laboratório, mas evite em produção

resource publicIP 'Microsoft.Network/publicIPAddresses@2022-09-01' = {
  name: '${vmName}-pip'
  location: location
  sku: {
    name: 'Basic'
  }
  properties: {
    publicIPAllocationMethod: 'Dynamic'
  }
}

resource vnet 'Microsoft.Network/virtualNetworks@2022-09-01' = {
  name: '${vmName}-vnet'
  location: location
  properties: {
    addressSpace: {
      addressPrefixes: [
        '10.0.0.0/16'
      ]
    }
    subnets: [
      {
        name: 'default'
        properties: {
          addressPrefix: '10.0.0.0/24'
        }
      }
    ]
  }
}

resource nic 'Microsoft.Network/networkInterfaces@2022-09-01' = {
  name: '${vmName}-nic'
  location: location
  properties: {
    ipConfigurations: [
      {
        name: 'ipconfig1'
        properties: {
          subnet: {
            id: vnet.properties.subnets[0].id
          }
          privateIPAllocationMethod: 'Dynamic'
          publicIPAddress: {
            id: publicIP.id
          }
        }
      }
    ]
  }
}

resource vm 'Microsoft.Compute/virtualMachines@2022-11-01' = {
  name: vmName
  location: location
  properties: {
    hardwareProfile: {
      vmSize: 'Standard_B1s' // Menor custo
    }
    osProfile: {
      computerName: vmName
      adminUsername: adminUsername
      adminPassword: adminPassword
    }
    storageProfile: {
      imageReference: {
        publisher: 'Canonical'
        offer: 'UbuntuServer'
        sku: '18.04-LTS'
        version: 'latest'
      }
      osDisk: {
        createOption: 'FromImage'
      }
    }
    networkProfile: {
      networkInterfaces: [
        {
          id: nic.id
        }
      ]
    }
  }
}
```

---

### 3. **Implantar o Bicep no Resource Group**

Execute o seguinte comando no terminal:

```bash
az deployment group create \
  --resource-group DIO-Lab-AZ104 \
  --template-file vm-lab.bicep
```

---






