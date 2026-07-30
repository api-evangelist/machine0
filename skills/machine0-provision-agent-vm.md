---
name: machine0-provision-agent-vm
description: Provision a persistent machine0 cloud VM for an AI agent, run a command on it, snapshot it, then suspend it to stop billing — using the machine0 remote MCP server.
api: machine0
transport: mcp
server: https://app.machine0.io/mcp
auth: OAuth 2.0 (interactive) or x-api-key header (programmatic)
operations:
  - size_list
  - ssh_key_create_managed
  - vm_create
  - vm_get_by_name
  - ssh_exec
  - image_create
  - vm_suspend
  - vm_start
  - vm_destroy
---

# Provision a persistent agent VM on machine0

Machine0 gives an agent a persistent, always-on cloud VM it can SSH into. Drive
it through the remote MCP server at `https://app.machine0.io/mcp`. Authenticate
interactively with OAuth, or set an `x-api-key` header for programmatic use.

## Steps

1. **Pick a size.** Call `size_list` to see available VM sizes with pricing and
   regional availability. Choose the smallest size that fits the workload (e.g.
   `small` for light agents, a `gpu-*` size only when GPU is required).

2. **Create a managed SSH key.** Call `ssh_key_create_managed` to create a
   server-managed keypair. This is required before you can run remote commands —
   `ssh_exec` only works on VMs provisioned with a managed key.

3. **Create the VM.** Call `vm_create` with a name, the chosen size, a region,
   and the managed SSH key. The VM stays running (and billed per minute) until
   you stop or suspend it.

4. **Confirm it is ready.** Call `vm_get_by_name` with the name you chose to read
   its status, static IP and HTTPS endpoint.

5. **Run work on it.** Call `ssh_exec` with the VM name and the command to run
   (optionally a timeout and a username override). Repeat as the agent needs.

6. **Snapshot before you tear down (optional).** Call `image_create` against the
   VM to capture its state as an image you can later restore from.

7. **Suspend to stop billing.** Call `vm_suspend` — this freezes VM state and
   stops compute billing (you keep paying only storage). Resume later with
   `vm_start`. Use `vm_destroy` instead when you want the VM gone permanently.

## Rules & conventions

- Billing is per-minute and continues while a VM is merely *stopped* — use
  `vm_suspend` (not stop) to actually stop compute charges. See
  `lifecycle/machine0-lifecycle.yml`.
- `ssh_exec` requires a managed key from `ssh_key_create_managed`; a
  user-registered public key alone will not authorize remote exec.
- Prefer `vm_get_by_name` for idempotent lookups by the name you assigned.
- Cross-reference: `mcp/machine0-mcp.yml` (full tool list),
  `authentication/machine0-authentication.yml`, `conventions/machine0-conventions.yml`.
