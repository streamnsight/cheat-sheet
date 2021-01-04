# Makefile tips and tricks

## loop through files

```Makefile
IN = $(wildcard inputs/*.txt)
OUT = $(subst inputs/,outputs/,$(IN))

outputs/%.txt: inputs/%.txt
        cp $< $@

default:  $(OUT)
```

