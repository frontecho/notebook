# Makefile

```makefile
obj : requirement
	[tab](@)Prompt

# Prompt前面加@，表示不打印这条命令

$@ $^ $<

= := += \(换行)

.PHONY : clean
```

```makefile
# 常用函数
$(func args)

cpp_srcs := $(shell find src -name "*.cpp")
cpp_objs := $(subst src/,objs/,$(cpp_objs))
cpp_objs := $(patsubst %.cpp,%.o,$(cpp_objs))

include_paths := /usr/include \
                 /home/include
library_paths := $(foreach item,$(include_paths),-L$(item))

I_flag := $(I_flag:%=**-I%**)

$(dir src/foo.c hacks) # 返回目录路径或者./
@mkdir -p $(dir $@)

libs := $(notdir $(libs)) # 输出文件名
alibs = $(fliter %.a,$(libs))
alibs = $(fliter-out %.so,$(libs))
alibs = $(basename $(alibs))
$(subst lib,,$(alibs))
```
