
## scope属性

+ compile: 默认范围，将依赖编译进class，同时也会被打包

+ provided：编译时需要，打包时不需要

+ runtime：编译时不需要，运行时需要

+ test：编译时不需要要，测试时需要

+ system：与provided类似，但是设置未system必须指定路径path

+ import: 在dependencyManagement使用，如当引入springboot依赖时，不想将它作为父依赖，可以单独指定只加载springboot依赖