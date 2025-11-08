# Crossplane Compositions: The Magic Behind Infrastructure as Code

## What is a Composition?
Imagine you're building a Lego set. Each Lego piece is a basic resource (like a VM or a network), but the instructions that tell you how to put them together to make a spaceship? That's a **Composition** in Crossplane!

A Composition is a YAML file that defines how to assemble multiple managed resources into a higher-level, reusable "composite resource." It's like a recipe that says: "To make a 'CompositeVM', you need a Virtual Machine, a Network Interface, maybe a Public IP, and here's how they connect." momkn 3la ay level t3ml composition bta3k zy level elinfra kamla.

## Composite Resource Definition (XRD)
Before you can use a Composition, you need to define the "API" for your composite resource. This is done with a **CompositeResourceDefinition (XRD)**. -> e3tbraha set of crds.

- **XRD** creates a new Kubernetes API resource (like `CompositeVM` in our lab).
- It specifies the schema: what fields your composite resource has (e.g., `vmSize`, `subnetName`, `public: true/false`).
- Once applied, you can create instances of this resource, and Crossplane will use the Composition to provision the underlying cloud resources.

In our lab, the XRD defines `CompositeVM` with specs like `vnetName`, `vmSize`, etc.

## How Compositions Work
Compositions use **functions** (like `patch-and-transform`) to manipulate data and create resources.

- **Pipeline Mode**: Our Composition uses `mode: Pipeline`, which allows chaining functions for complex logic.
- **Resources**: Each resource in the Composition (e.g., `nic` for NetworkInterface, `linuxvm` for LinuxVirtualMachine) is defined with a `base` template and `patches` to customize it.
- **Patches**: These copy values from the composite resource spec to the managed resources. For example, `fromFieldPath: spec.vmSize` patches the VM size.
- **Transforms**: Modify data on the fly, like formatting strings (e.g., building subnet names).
- **Readiness Checks**: Ensure resources are ready before proceeding (e.g., check `provisioningState: Succeeded`).

## Why Use Compositions?
- **Abstraction**: Hide cloud complexity. Users just say "I want a VM with these specs," and Crossplane handles the networking, security, etc.
- **Reusability**: One Composition can be used for multiple instances (e.g., admin VM, portal VM).
- **Consistency**: Ensures all resources follow the same patterns and best practices.
- **GitOps Friendly**: Define infrastructure as code, version it, and apply via kubectl.

## composition functions
functions are the building blocks of compositions, allowing you to manipulate and transform data as needed. -> lw hn3ml loop 3la resources mo3ayna masln aw patch and transform data from the composite resource to coposition and more ...

## Our Lab's Composition Breakdown
- **01-composition-compositevm.yaml**: The main Composition for `CompositeVM`.
  - Creates a NIC (Network Interface) with subnet and optional public IP.
  - Creates a Linux VM attached to the NIC.
  - Uses patches to inject values from the CompositeVM spec.
- **07-compositevm-instances.yaml**: Uses Helm templating to generate multiple `CompositeVM` instances from `values.yaml`.

This setup lets you deploy a full lab environment (VMs, network, app services) with a single Helm install!
