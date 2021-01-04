# Networking tips and tricks

## Tunneling a service through a bastion host

```bash
export BASTION_IP=
export RHOST=
export PORT=
# Create tunnel
ssh -M -S socket -fnNT -L ${PORT}:${RHOST}:${PORT} opc@${BASTION_IP} cat -
ssh -S socket -O check opc@${BASTION_IP}

# close the tunnels
ssh -S socket -O exit opc@${BASTION_IP}

# remove entry in known_hosts so it can be reused 
ssh-keygen -R [localhost]:${PORT}
```

## SCP proxy

```bash
export BASTION_IP=
export RHOST=
export PORT=
scp -o ProxyCommand="ssh -W %h:%p opc@${BASTION_IP} file.ext opc@${RHOST}:~/
```

## SSH or SCP with proxy

```bash
PROXY_CMD="-o ProxyCommand=\"ssh opc@${BASTION_IP} -W %h:%p\""

function ssh_ {
    CMD=(ssh "${PROXY_CMD}" "$1" """$2""")
    echo "Executing: ${CMD[@]}"
    eval "${CMD[@]}"
}
function scp_ {
    CMD=(scp "${PROXY_CMD}" "$1" "$2")
    echo "Executing: ${CMD[@]}"
    eval "${CMD[@]}"
}
```

example:
```bash
BASTION_IP=
# source the script
scp_ "source.*" opc@${TARGET_IP}:~/
ssh_ opc@${TARGET_IP} "sudo do-something" 
```

## SSH tunnel

```bash
ssh -J opc@${BASTION_IP} opc@${TARGET_IP}
```

## SOCKS proxy

```bash
ssh -C -D $PORT -i <private_ssh-key> opc@${BASTION_IP}
```

Then configure browser SOCKS proxy and connect to localhost on $PORT

## Find open ports on machine

```
sudo lsof -i -P -n | grep LISTEN
```

