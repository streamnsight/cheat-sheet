# wget tips and tricks

## Download a full website with its content

```bash
wget -r -p -U Mozilla --wait=10 --limit-rate=35K https://www.makeuseof.com
```

-r recursive
-p download assets (images etc..)
-U user-agent
-w wait time 
—limit-rate 

