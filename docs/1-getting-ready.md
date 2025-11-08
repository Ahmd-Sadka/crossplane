# Getting Ready: Your Crossplane Lab Setup Adventure 🛠️🚀

Ahoy, fellow cloud adventurer! Before we unleash the power of Crossplane to conjure up Azure infrastructure like a wizard casting spells, let's get your toolkit ready. Think of this as prepping your spaceship before blasting off to the stars – we need fuel (tools), coordinates (credentials), and a map (configs). 

## Prerequisites: The Essentials You Can't Skip
Before we dive in, grab these tools. They're like the Avengers of your dev setup – each has a superpower!

1. **Kubernetes Cluster**: Your control center. Use Minikube for local fun, or a cloud cluster (AKS, EKS) for the real deal. Make sure `kubectl` is pointing to it.
2. **Azure CLI**: Your Azure whisperer. Install it and log in – it's how we chat with Azure.
3. **Crossplane CLI**: Crossplane's sidekick. Grab it from [here](https://docs.crossplane.io/install.sh) and run the install script.
4. **Helm**: The package manager extraordinaire. Version 3+ is your friend for deploying charts.

Pro Tip: If you're on Linux, these install like a breeze. Test with `az --version`, `crossplane --version`, etc. If something's missing, Google it.

## Step-by-Step Setup: Let's Build This Rocket! 🚀

### Step 1: Install Crossplane in Your K8s Cluster
Crossplane is the engine that makes infrastructure-as-code magic happen. Install it via Helm:

```bash
helm repo add crossplane-stable https://charts.crossplane.io/stable
helm repo update
helm install crossplane crossplane-stable/crossplane --namespace crossplane-system --create-namespace --wait
```

Verify it's alive:
```bash
kubectl get pods -n crossplane-system
```
You should see pods running happily. If not, check logs with `kubectl logs -n crossplane-system deployment/crossplane`. No pods? Your cluster might be grumpy – double-check permissions!

### Step 2: Create an Azure Service Principal (SP)
Crossplane needs Azure creds to boss around resources. Think of the SP as a VIP pass for Crossplane to enter Azure's exclusive club.

```bash
az login  # Log in to Azure first
az ad sp create-for-rbac --name crossplane-sp --role Contributor --scopes /subscriptions/YOUR_SUBSCRIPTION_ID --sdk-auth > azure-creds.json
```

**YOUR_SUBSCRIPTION_ID**: Grab this from `az account list`. Store `azure-creds.json` like it's your grandma's secret cookie recipe – securely, not in Git! 🔒

### Step 3: Create a Kubernetes Secret for Azure Creds
Now, smuggle those creds into K8s as a secret. Crossplane will use this to authenticate.

```bash
kubectl create secret generic azure-secret --namespace crossplane-system --from-file=creds=./azure-creds.json
```

Check it:
```bash
kubectl get secret azure-secret -n crossplane-system
```
If it's there, high-five yourself! 🙌

### Step 4: Apply the Provider Config
This tells Crossplane: "Hey, use these creds for Azure stuff."

```bash
kubectl apply -f provider-config.yaml
```

It creates a `ProviderConfig` named `default` that points to our secret. Simple, right? Like telling your GPS where home is.

### Step 5: Install the Azure Providers
Providers are Crossplane's plugins for cloud services. We use modular ones (network, compute, web) instead of the old bulky one – think of it as upgrading from a clunky van to sleek sports cars! 🏎️

```bash
kubectl apply -f providers/
```

Wait for them to be healthy:
```bash
kubectl get providers
```
Status should say "Healthy." If not, patience is key – or check `kubectl describe provider YOUR_PROVIDER`.

Why modular? The old provider was like a kitchen sink with 1000+ CRDs. Modular keeps it lean and mean, updating faster than fashion trends.

### Bonus: Apply RBAC and XRD
For extra permissions and to define our custom `CompositeVM` resource:

```bash
kubectl apply -f crossplane-compositevm-rbac.yaml
kubectl apply -f crossplane-compositevm-rbac-binding.yaml
kubectl apply -f 00-compositevm-xrd.yaml
```

Now you're set to deploy the lab!

## Troubleshooting: When Things Go Boom 💥
- **Crossplane not installing?** Check Helm repos and K8s permissions.
- **Providers unhealthy?** Logs are your friends: `kubectl logs -n crossplane-system provider-YOUR_PROVIDER`.
- **Auth fails?** Verify the secret and SP. Test with `az login --service-principal -u SP_APP_ID -p SP_PASSWORD --tenant TENANT_ID`.
- **Stuck resources?** Use `kubectl describe` on failing resources for clues.

Remember: Debugging is like being a detective – follow the breadcrumbs!

## What's Next? The Grand Finale! 🎉
With this setup, deploy the lab:
```bash
helm install ndc-env-lab ./Env-Lab
```
Watch your Azure empire rise! Modify `values.yaml` for customizations, and redeploy with `helm upgrade`. For full deployment vibes, check `docs/deployment-guide.md`.

Questions? The Crossplane community is awesome – hit up their Slack or docs. You've got this! 🌟
