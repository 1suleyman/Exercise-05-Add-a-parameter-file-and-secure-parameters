# 🗂️ Using Parameter Files in Bicep — Keep It Clean, Keep It Scalable

> **Bitesize Lesson 🍬**  
You've been passing parameter values directly on the CLI like a champ — but once your templates grow, it's time to *scale up* your game.  
Enter: **parameter files**. These make your deployments more maintainable, organized, and automation-friendly.  
Also, hi again, future-me 👋 — this is why you didn’t just jam everything into the command line.

---

## 🧾 What’s a Parameter File?

A **parameter file** is a tidy way to group all your parameter values together, especially when:
- You have **lots of parameters**
- You’re deploying to **multiple environments**
- You want to keep your CLI commands clean  
- You’re automating deployments via pipelines

You can use:
- `.bicepparam` files (native Bicep)
- JSON files (classic, still popular)

---

## 🧠 JSON Parameter File Example

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "appServicePlanInstanceCount": {
      "value": 3
    },
    "appServicePlanSku": {
      "value": {
        "name": "P1v3",
        "tier": "PremiumV3"
      }
    },
    "cosmosDBAccountLocations": {
      "value": [
        { "locationName": "australiaeast" },
        { "locationName": "southcentralus" },
        { "locationName": "westeurope" }
      ]
    }
  }
}
```

🧩 Match the keys to the parameter names in your `.bicep` file  
✅ Values go under `"value"`  
📄 Save it as something like `main.parameters.dev.json`

---

## 🚀 Deploying with a Parameter File

Instead of this:

```bash
az deployment group create \
  --name main \
  --template-file main.bicep \
  --parameters appServicePlanInstanceCount=3 ...
```

Use this:

```bash
az deployment group create \
  --name main \
  --template-file main.bicep \
  --parameters main.parameters.dev.json
```

So much cleaner, right?

---

## ⚖️ What If You Need to Override Something?

You can **override values** in the parameters file by also specifying them on the command line:

```bash
az deployment group create \
  --template-file main.bicep \
  --parameters main.parameters.dev.json \
               appServicePlanInstanceCount=5
```

Azure follows this **order of precedence**:

```
Default (in .bicep) < Parameter file < Command line
```

So you can:
- Set safe defaults for dev/test
- Use param files for consistency across environments
- Still override stuff manually when needed

---

## 📦 Parameter File Best Practices

✅ **Use one file per environment**  
Name them clearly:
```
main.parameters.dev.json
main.parameters.prod.json
```

✅ **Only define parameters that exist in your Bicep file**  
Extra parameters = errors at deployment time ❌

✅ **Avoid putting secrets here**  
Use Key Vault instead (more on that in a later lesson!)

---

## 🧪 Real-World Example

Here’s a `.bicep` file with defaults:

```bicep
param location string = resourceGroup().location
param appServicePlanInstanceCount int = 1
param appServicePlanSku object = {
  name: 'F1'
  tier: 'Free'
}
```

Here’s a JSON file that overrides **some** of those:

```json
{
  "parameters": {
    "appServicePlanInstanceCount": { "value": 3 },
    "appServicePlanSku": {
      "value": { "name": "P1v3", "tier": "PremiumV3" }
    }
  }
}
```

And a CLI command that overrides *even that*:

```bash
--parameters main.parameters.dev.json \
             appServicePlanInstanceCount=5
```

🔍 Final values:
- `location`: stays default from the template
- `sku`: comes from the file
- `instanceCount`: overridden by CLI to `5`

---

## 🎯 TL;DR Recap

- 🧾 Parameter files keep deployments organized
- 📁 Create one per environment (`dev`, `test`, `prod`)
- 🧼 Use them to avoid long CLI commands
- 🧠 CLI > Param File > Default (for precedence)
- 🛡️ Don’t put secrets in param files (use Key Vault!)
