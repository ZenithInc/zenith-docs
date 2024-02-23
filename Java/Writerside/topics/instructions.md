# 指令集概览

一条指令集包含了一个操作，由一个字节的操作码和零到多个的操作数组成。大多数的操作码并没有参数。下表展示了一个 opcode 在不同的类型下的不同指令。

| **opcode** | **byte** | **short** | **int** | **long** | **float** | **double** | **char** | **reference** |
|------------|----------|-----------|---------|----------|-----------|------------|----------|---------------|
| Tipush     | bipush   | sipush    |         |          |           |            |          |               |
| Tconst     |          |           | iconst  | lconst   | fconst    | dconst     |          | aconst        |
| Tload      |          |           | iload   | lload    | fload     | dload      |          | aload         |
| Tstore     |          |           | istore  | lstore   | fstore    | dstore     |          | astore        |
| Tinc       |          |           | iinc    |          |           |            |          |               |
| Taload     | baload   | saload    | iaload  | laload   | faload    | daload     | caload   | aaload        |
| Tastore    | bastore  | sastore   | iastore | lastore  | fastore   | dastore    | castore  | aastore       |
| Tadd       |          |           | iadd    | ladd     | fadd      | dadd       |          |               |
| Tsub       |          |           | isub    | lsub     | fsub      | dsub       |          |               |

<note>
opcode 在规范中也称之为“助记符”，是给程序员看的。在 JVM 中识别的是助记符所代表的二进制指令。
</note>

我们可以通过上面这张从规范章截取的表格中看到，相同的操作在不同的数据类型下有着不同的指令。在 `opcode` 中，`T` 表示类型，在使用的时候替换成指定的类型。比如说同样的操作 `ipush` ，在针对 `byte` 类型操作的时候，使用的指令是 `bipush`, 而针对 `short` 类型操作的时候，指令时 `sipush`。