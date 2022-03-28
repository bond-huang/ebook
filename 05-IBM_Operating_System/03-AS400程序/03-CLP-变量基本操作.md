# CLP-变量基本操作
&#8195;&#8195;变量是一个命名的可变值，可以通过引用其名称来访问或更改，变量不存储在库中，它们也不是对象。官方文档主页：[IBM i 7.3 CL 命令中的变量](https://www.ibm.com/docs/zh/i/7.3?topic=programming-variables-in-cl-commands)
## 定义变量
所有变量必须先向CL程序或过程声明（定义），然后才能被程序或过程使用。
### 声明变量的方法
有两种声明变量的方法：Declare variable及Declare file：
- 声明变量：使用`DCL`(Declare CL Variable)命令完成的，包括定义变量的属性。这些属性包括类型、长度和初始值:
    ```
    DCL  VAR(&AREA)  TYPE(*CHAR)  LEN(4)  VALUE(BOOK)
    DCL  VAR(&TMPDATE) TYPE(*CHAR) LEN(8) 
    DCL  VAR(&DATE) TYPE(*CHAR) LEN(6) 
    ```
- 声明文件：如果CL程序或过程使用文件，则必须在`DCLF`(Declare File)命令的`FILE`参数中指定文件的名称。该文件包含文件中记录的描述（格式）和记录中的字段:
    ```
    DCLF  FILE(MCGANN/GUIDE)
    ```
    - 在编译期间，DCLF 命令为文件中定义的字段和指标隐式声明CL变量。例如，如果文件的DDS中有一个记录，其中包含两个字段（F1和F2），则程序中会自动声明两个变量`&F1`和`&F2`
    - 如果文件是在没有DDS的情况下创建的物理文件，则为整个记录声明一个变量。变量与文件同名，长度与文件的记录长度相同

声明命令必须在程序或过程中的所有其他命令之前（除了PGM命令），但它们可以以任何顺序混合。
### 使用声明CL变量命令的规则
&#8195;&#8195;`DCL`(Declare CL Variable)命令官方链接：[IBM i 7.3 Declare CL Variable (DCL)](https://www.ibm.com/docs/zh/i/7.3?topic=ssw_ibm_i_73/cl/dcl.htm)。使用`DCL`命令时，必须使用以下规则：
- CL变量名称必须以与号`&`开头，后跟最多10个字符，并且第一个字符必须是字母，其余字符必须是字母数字。例如`&TESTVAR`
- CL变量值必须是以下之一：
    - 长达5000个字符的字符串
    - 一个压缩十进制值，总计最多15位，最多9个小数位
    - 逻辑值`0`或`1`，其中`0`表示`off`、`false`或`no`, `1`表示`on`、`true`或`yes`
    - 两个、四个或八个字节的整数值:如果为`TYPE`参数指定了`*INT`，则该值可以是负数；如果为`TYPE`参数指定了 `*UINT`，则该值必须为正或零。仅当使用`CRTCLMOD`(Create CL Module)命令或`CRTBNDCL`(Create Bound CL Program)命令编译CL源时，才能指定`LEN(8)`
    - 一个指针值，可以保存数据在存储中的位置
- 如果不指定初始值，则默认如下：
    - `0`表示十进制变量
    - 空白的字符变量
    - `0`表示逻辑变量
    - `0`表示整数变量
    - 指针变量为空
    - 对于十进制和字符类型，如果指定了初始值，不指定`LEN`参数，则默认长度与初始值的长度相同
    - 对于`*CHAR`类型，如果不指定`LEN`参数，则字符串可以长达5000个字符
    - 对于`*INT`或`*UINT`类型，如果不指定`LEN`参数，则默认长度为4
- 在程序 DCL 语句中将参数声明为变量

## 基本变量用途
&#8195;&#8195;基于变量可用于映射传递给程序的变量或操作值数组。在使用之前，必须使用`DCL`命令上的`ADDRESS`关键字或使用`%ADDRESS`内置函数来设置基础指针。设置基础指针后，变量将像局部变量一样工作。示例：
```
PGM
    DCL &AUTO  *CHAR 20
    DCL &PTR   *PTR ADDRESS(&AUTO)
    DCL &BASED *CHAR 10 STG(*BASED) BASPTR(&PTR) 
    :
    IF COND(%SST(&AUTO 1 10) *EQ &BASED) +
        THEN(CHGVAR %OFS(&PTR) (%OFS(&PTR) + 10))
    :
ENDPGM
```
示例说明：
- 示例中，基础指针`&PTR`被声明为等于`&AUTO`的地址。然后变量`&BASED`具有指针变量`&PTR`寻址的前10个字节的值
- 在条件语句中，将检查变量`&BASED`的值与变量`&AUTO`的前10个字节是否相等。如果值相同，意味着指针`&PTR`寻址`&AUTO`的第一个字节，则指针偏移量更改为变量`&AUTO`的地址字节11
- 现在变量`&BASED`的值等于变量`&AUTO`的11-20字节

### 定义变量用途
