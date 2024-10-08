---
search:
  exclude: true
---


!!! abstract 

    [样式测试](https://squidfunk.github.io/mkdocs-material/reference/formatting/#configuration)



## 注记

支持 attention, caution, danger, error, hint, important, note, tip, and warning

若要用标题，可以在 attention 等后面加上"标题参数"。如：`!!! abstract "标题"`

1. 只有图标

    === "效果"

        !!! abstract
            简介

    === "代码"

        ```markdown
        !!! abstract
            简介

        ```
        
2. 可展开

    === "效果"

        ??? abstract
            简介

    === "代码"

        ```markdown
        ??? abstract
            简介

        ```

3. 默认展开

    === "效果"

        ???+ note
            笔记

    === "代码"

        ```markdown
        ???+ note
            笔记

        ```

## 标签页

=== "C"

    ``` c
    #include <stdio.h>

    int main(void) {
      printf("Hello world!\n");
      return 0;
    }
    ```

=== "C++"

    ``` c++
    #include <iostream>

    int main(void) {
      std::cout << "Hello world!" << std::endl;
      return 0;
    }
    ```
## 按键

=== "效果"

    ++ctrl+alt+del++

=== "代码"

    ```markdown
        ++ctrl+alt+del++

    ```

## markmap

=== "效果"

{!mindmap.mm.md!}


??? note
    
    ```markdown
        ```markmap
        ---
        markmap:
            colorFreezeLevel: 2
        ---
            # Root

            ## Branch 1

            * Branchlet 1a
            * Branchlet 1b

            ## Branch 2

            * Branchlet 2a
            * Branchlet 2b
        ```  
    ```

    或，直接引入文件：

    ```markdown

    <!-- 请删除"\" -->
    \{!mindmap.mm.md!\}

    ``` 





## 恢复测试

测试测试测试
测试测试测试