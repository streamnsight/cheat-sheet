# shell scripting tips and tricks

## Substitute environment variables in a file

```bash
envsubst < input.yml > output.yml
```

## Multiline input / output

### print multiline to file
```bash
cat > file <<EOF

EOF
```

### Send multiline string to command
```bash
cat | wc -l <<EOF

EOF
```
