makefile中使用
``` C
define KCL_MACRO_CHECK_COMMAND  
$(shell if ! [ -f $(config-file) ] || grep -q "#define $(1)" $(config-file) ; \
    then echo "Error: Symbol $(1) is found in $(config-file)"; fi)
endef
```
由于换行符 `\` 后跟有空格导致编译错误，推测shell有强格式要求