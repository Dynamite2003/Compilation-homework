1.验证如下文法是LL(1)文法。若是，则请构造递归下降分析程序。
E → A a | B b
A → c A | e B
B → b d

要验证给定的文法是否是LL(1)文法，并在确认后构造相应的递归下降分析程序，我们可以按照以下步骤进行。

### 给定文法

```
E → A a | B b
A → c A | e B
B → b d
```

### 步骤1：计算FIRST集

- **FIRST(A)**:
  - A → c A | e B
  - FIRST(A) = {c, e}

- **FIRST(B)**:
  - B → b d
  - FIRST(B) = {b}

- **FIRST(E)**:
  - E → A a | B b
  - FIRST(A a) = FIRST(A) = {c, e}
  - FIRST(B b) = FIRST(B) = {b}
  - 因此，FIRST(E) = {c, e, b}

### 步骤2：计算FOLLOW集

假设E是开始符号，因此FOLLOW(E)包含文件结束符号 `$`。

- **FOLLOW(E)** = { $ }

- **FOLLOW(A)**:
  - 从生产式 E → A a，可以得出FOLLOW(A)包含 {a}

- **FOLLOW(B)**:
  - 从生产式 E → B b，可以得出FOLLOW(B)包含 {b}
  - 从生产式 A → e B，由于B在产生式的末尾，FOLLOW(B)还包含FOLLOW(A)，即 {a}

综上：

- FOLLOW(E) = { $ }
- FOLLOW(A) = { a }
- FOLLOW(B) = { b, a }

### 步骤3：验证LL(1)条件

为了确认文法是LL(1)，需要确保每个非终结符的不同产生式的FIRST集互不相交。如果某些产生式可以推导出ε（空串），还需要确保它们的FIRST集与FOLLOW集互不相交。

- **对于E**:
  - 产生式 E → A a 和 E → B b
  - FIRST(A a) = {c, e}
  - FIRST(B b) = {b}
  - {c, e} 与 {b} 不相交。

- **对于A**:
  - 产生式 A → c A 和 A → e B
  - FIRST(c A) = {c}
  - FIRST(e B) = {e}
  - {c} 与 {e} 不相交。

- **对于B**:
  - 只有一个产生式 B → b d
  - 无冲突。

由于没有任何产生式能够推导出ε，因此不需要进一步检查FOLLOW集。

因此，给定的文法是LL(1)文法。

### 步骤4：构造递归下降分析程序

递归下降分析器为每个非终结符定义一个函数，通过查看当前的输入符号（预读符号）来决定使用哪条产生式。以下是基于Python的递归下降分析器的示例实现：

```python
class Parser:
    def __init__(self, tokens):
        self.tokens = tokens  # 输入的token列表
        self.pos = 0
        self.current_token = self.tokens[self.pos]

    def next_token_func(self):
        """移动到下一个token"""
        self.pos += 1
        if self.pos < len(self.tokens):
            self.current_token = self.tokens[self.pos]
        else:
            self.current_token = 'EOF'  # 表示输入结束

    def parse(self):
        """开始解析"""
        self.E()
        if self.current_token != 'EOF':
            self.error("解析结束后还有多余的输入。")

    def E(self):
        """解析E"""
        if self.current_token in ['c', 'e', 'b']:
            if self.current_token in ['c', 'e']:
                self.A()
                self.match('a')
            elif self.current_token == 'b':
                self.B()
                self.match('b')
        else:
            self.error("E预期遇到 'c', 'e' 或 'b'。")

    def A(self):
        """解析A"""
        if self.current_token == 'c':
            self.match('c')
            self.A()
        elif self.current_token == 'e':
            self.match('e')
            self.B()
        else:
            self.error("A预期遇到 'c' 或 'e'。")

    def B(self):
        """解析B"""
        if self.current_token == 'b':
            self.match('b')
            self.match('d')
        else:
            self.error("B预期遇到 'b'。")

    def match(self, token):
        """匹配当前token，并移动到下一个token"""
        if self.current_token == token:
            print(f"匹配: {token}")
            self.next_token_func()
        else:
            self.error(f"预期 token '{token}', 但遇到 '{self.current_token}'。")

    def error(self, message):
        """抛出解析错误"""
        raise Exception(f"解析错误在位置 {self.pos}: {message}")

# 示例使用：
# 假设输入token序列为 ['c', 'c', 'e', 'b', 'd', 'a', 'EOF']
tokens = ['c', 'c', 'e', 'b', 'd', 'a', 'EOF']
parser = Parser(tokens)
try:
    parser.parse()
    print("解析成功！")
except Exception as e:
    print(str(e))
```

### 解释

1. **类 `Parser`**:
    - 初始化时接收一个token列表，并设置当前token为第一个token。
    - `next_token_func` 方法用于移动到下一个token，如果到达输入末尾，则设置为 `'EOF'`。

2. **方法 `parse`**:
    - 从开始符号 `E` 开始解析。
    - 解析完成后，检查是否已经到达输入的结束标记 `'EOF'`，如果还有多余的输入则报错。

3. **方法 `E`**:
    - 根据当前token决定使用哪条产生式：
        - 如果当前token是 `c` 或 `e`，则使用产生式 `E → A a`，调用 `A()`，然后匹配 `a`。
        - 如果当前token是 `b`，则使用产生式 `E → B b`，调用 `B()`，然后匹配 `b`。
    - 其他情况下报错。

4. **方法 `A`**:
    - 根据当前token决定使用哪条产生式：
        - 如果当前token是 `c`，则使用产生式 `A → c A`，匹配 `c` 后递归调用 `A()`。
        - 如果当前token是 `e`，则使用产生式 `A → e B`，匹配 `e` 后调用 `B()`。
    - 其他情况下报错。

5. **方法 `B`**:
    - 使用产生式 `B → b d`，需要匹配 `b` 和 `d`。
    - 如果当前token不是 `b`，则报错。

6. **方法 `match`**:
    - 检查当前token是否与预期token匹配，如果匹配则移动到下一个token，否则报错。

7. **方法 `error`**:
    - 抛出解析错误，指出出错的位置和原因。

### 示例

假设输入的token序列为 `['c', 'c', 'e', 'b', 'd', 'a', 'EOF']`，解析过程如下：

1. 调用 `E()`:
    - 当前token是 `'c'`，属于 `FIRST(A a)`，选择 `E → A a`。
    - 调用 `A()`:
        - 当前token是 `'c'`，选择 `A → c A`，匹配 `'c'`，递归调用 `A()`:
            - 当前token是 `'c'`，选择 `A → c A`，匹配 `'c'`，再次递归调用 `A()`:
                - 当前token是 `'e'`，选择 `A → e B`，匹配 `'e'`，调用 `B()`:
                    - 当前token是 `'b'`，选择 `B → b d`，匹配 `'b'` 和 `'d'`。
    - 回到 `E()`，匹配 `'a'`。

2. 最后，检查到达 `'EOF'`，解析成功。

输出将显示匹配过程并最终确认解析成功。


2. 验证如下文法是LL(1)文法。若是，则请构造文法的LL(1)分析表。
D → T L
T → int | real
L → id R
R → , id R | ε
要验证给定的文法是否为LL(1)文法，并在确认后构造其LL(1)分析表，我们将按照以下步骤进行：

1. **计算FIRST集和FOLLOW集**
2. **验证LL(1)条件**
3. **构造LL(1)分析表**

### 给定文法

```
D → T L
T → int | real
L → id R
R → , id R | ε
```

### 步骤1：计算FIRST集和FOLLOW集

首先，我们需要确定每个非终结符的FIRST集和FOLLOW集。

#### 1.1 FIRST集

**定义**：对于任意符号串α，FIRST(α)是可以出现在由α推导出的字符串的第一个终结符的集合。

- **FIRST(D)**：
  - D → T L
  - FIRST(D) = FIRST(T)

- **FIRST(T)**：
  - T → int | real
  - FIRST(T) = { `int`, `real` }

- **FIRST(L)**：
  - L → id R
  - FIRST(L) = { `id` }

- **FIRST(R)**：
  - R → , id R | ε
  - FIRST(R) = { `,`, ε }

综上：

- **FIRST(D) = { `int`, `real` }**
- **FIRST(T) = { `int`, `real` }**
- **FIRST(L) = { `id` }**
- **FIRST(R) = { `,`, ε }**

#### 1.2 FOLLOW集

**定义**：对于非终结符A，FOLLOW(A)是所有能跟在A之后的终结符的集合。

**规则**：
1. 将 `$`（输入结束符）加入到开始符号的FOLLOW集中。
2. 对于每个产生式 `A → α B β`，将FIRST(β) \ {ε} 加入到FOLLOW(B)。
3. 对于每个产生式 `A → α B` 或 `A → α B β`，如果 `β` 能推导出ε（即FIRST(β)包含ε），则将FOLLOW(A)加入到FOLLOW(B)。

**分析**：

- 假设 **D** 是开始符号，因此：

  - **FOLLOW(D) = { `$` }**

- **D → T L**：

  - 对于 **T**：
    - T 后跟 L
    - FIRST(L) = { `id` }
    - 因此，`id` 加入到 **FOLLOW(T)**

  - 对于 **L**：
    - L 后没有符号，因此 FOLLOW(L) 包含 FOLLOW(D) = { `$` }
    - 因此，`$` 加入到 **FOLLOW(L)**

- **T → int | real**：
  - 终结符的FOLLOW集无需扩展

- **L → id R**：

  - 对于 **id**：
    - id 后跟 R
    - FIRST(R) = { `,`, ε }
    - 因此，``,` 加入到 FOLLOW(id)`，但id是终结符，不作为非终结符，因此不需要考虑。

  - 对于 **R**：
    - R 后没有符号，因此 FOLLOW(R) 包含 FOLLOW(L) = { `$` }
    - 因此，`$` 加入到 **FOLLOW(R)**

- **R → , id R | ε**：

  - 对于 **, id R**：
    - `,` 是终结符，FOLLOW集无需扩展

  - 对于 ε：
    - 由于R可以推导出ε，FOLLOW(R) = { `$` }

综上：

- **FOLLOW(D) = { `$` }**
- **FOLLOW(T) = { `id` }**
- **FOLLOW(L) = { `$` }**
- **FOLLOW(R) = { `$` }**

### 步骤2：验证LL(1)条件

**LL(1)条件**要求，对于每个非终结符的不同产生式，其FIRST集必须互不相交。如果某些产生式能推导出ε，还需确保其FIRST集与FOLLOW集不相交。

**逐个非终结符检查**：

1. **D → T L**
   - 只有一个产生式，因此无需冲突。

2. **T → int | real**
   - FIRST(`int`) = { `int` }
   - FIRST(`real`) = { `real` }
   - `{ int }` 与 `{ real }` 不相交。

3. **L → id R**
   - 只有一个产生式，因此无需冲突。

4. **R → , id R | ε**
   - FIRST(`, id R`) = { `,` }
   - FIRST(ε) = { ε }
   - `{ , }` 与 `{ ε }` 不相交。
   - 检查ε的情况：确保 FIRST(`R → , id R`) ∩ FOLLOW(R) = { `,` } ∩ { `$` } = ∅，满足条件。

由于所有非终结符满足LL(1)条件，因此该文法是LL(1)文法。

### 步骤3：构造LL(1)分析表

**步骤**：

1. **确定终结符和非终结符**：

   - **非终结符**： D, T, L, R
   - **终结符**： int, real, id, ,, $
   
2. **构造表格**：

   表的行表示非终结符，列表示终结符（包括 `$` ）。每个单元格填入相应的产生式。

   | 非终结符 | `int`   | `real`   | `id`     | `,`        | `$`   |
   | -------- | ------- | -------- | -------- | ---------- | ----- |
   | **D**    | D → T L | D → T L  |          |            |       |
   | **T**    | T → int | T → real |          |            |       |
   | **L**    |         |          | L → id R |            |       |
   | **R**    |         |          |          | R → , id R | R → ε |

**详细说明**：

- **D**：
  - FIRST(D) = { `int`, `real` }
  - 所以在 `int` 和 `real` 列下，填写 D → T L。

- **T**：
  - T → int 在 `int` 列下。
  - T → real 在 `real` 列下。

- **L**：
  - FIRST(L) = { `id` }
  - 所以在 `id` 列下，填写 L → id R。

- **R**：
  - R → , id R 在 `,` 列下。
  - R → ε 需要在FOLLOW(R)的符号下填写，即在 `$` 列下填写 R → ε。

**完整的LL(1)分析表**：

| 非终结符 | `int`   | `real`   | `id`     | `,`        | `$`   |
| -------- | ------- | -------- | -------- | ---------- | ----- |
| **D**    | D → T L | D → T L  |          |            |       |
| **T**    | T → int | T → real |          |            |       |
| **L**    |         |          | L → id R |            |       |
| **R**    |         |          |          | R → , id R | R → ε |

**注意**：

- 表格中的空白表示在该终结符下无对应的产生式，若在解析过程中遇到这种情况，意味着语法错误。
- `$` 表示输入结束符，用于检测输入是否完全匹配文法。


3.消除如下文法的左递归 
A → Ba | Aa | c
B → Bb | Ab | d

要消除给定文法中的左递归，我们需要遵循系统的步骤来确保文法转换后仍保持等价，并且不含有左递归。以下是详细的步骤和最终消除左递归后的文法。

### 给定文法

```
A → Ba | Aa | c
B → Bb | Ab | d
```

### 步骤1：识别左递归

**左递归**指的是一个非终结符在其产生式中直接或间接地以自身开始。例如：

- **直接左递归**：`A → Aa`
- **间接左递归**：如果 `A → Bα` 且 `B → Aβ`，则存在间接左递归。

在给定的文法中：

- **A** 有直接左递归：`A → Aa`
- **B** 有直接左递归：`B → Bb`
- **A** 和 **B** 之间存在间接左递归：
  - `A → Ba`
  - `B → Ab`

### 步骤2：确定处理顺序

为了系统地消除左递归，通常按照一定的顺序处理非终结符。我们选择按字母顺序处理：

1. **A**
2. **B**

### 步骤3：消除左递归

#### 3.1 消除 **A** 的左递归

**A 的产生式**：
```
A → Ba | Aa | c
```

**识别左递归**：
- 左递归产生式：`A → Aa`
- 非左递归产生式：`A → Ba | c`

**转换方法**：

按照消除直接左递归的标准方法，重写 **A** 的产生式：

```
A → Ba A’ | c A’
A’ → a A’ | ε
```

**解释**：
- `A’` 是一个新的非终结符，用于处理递归部分。
- `A → Ba A’ | c A’` 取代了原来的 `A → Ba | Aa | c`。
- `A’ → a A’ | ε` 处理原来的 `Aa` 递归和允许结束递归。

#### 3.2 消除 **B** 的左递归

**B 的产生式**：
```
B → Bb | Ab | d
```

**处理顺序**：
由于 **A** 在 **B** 之前处理，我们需要替换 **B** 中使用 **A** 的产生式。

**替换 `A`**：
从 **A** 的转换结果：
```
A → Ba A’ | c A’
```
将 `A` 替换到 **B** 的产生式中：

```
B → Bb | (Ba A’) b | (c A’) b | d
```

简化后：
```
B → Bb | Ba A’ b | c A’ b | d
```

**识别左递归**：
- 左递归产生式：`B → Bb | Ba A’ b`
- 非左递归产生式：`B → c A’ b | d`

**转换方法**：

同样使用消除直接左递归的方法，引入新的非终结符 `B’`：

```
B → c A’ b B’ | d B’
B’ → b B’ | a A’ b B’ | ε
```

**解释**：
- `B’` 处理 **B** 的递归部分。
- `B → c A’ b B’ | d B’` 取代了原来的 `B → Bb | Ba A’ b | c A’ b | d`。
- `B’ → b B’ | a A’ b B’ | ε` 处理原来的 `Bb` 和 `Ba A’ b` 递归，同时允许结束递归。

### 步骤4：最终消除左递归后的文法

综合上述步骤，消除左递归后的文法如下：

```
A → Ba A’ | c A’
A’ → a A’ | ε

B → c A’ b B’ | d B’
B’ → b B’ | a A’ b B’ | ε
```

### 步骤5：验证文法

确保转换后的文法不含有左递归，并且保留了原始文法的语言生成能力。

- **A 的产生式**：
  - `A` 不再以自身开头，左递归已消除。
  
- **B 的产生式**：
  - `B` 不再以自身开头，左递归已消除。

### 结论

通过系统地应用消除左递归的方法，我们成功地将原始文法转换为不含左递归的等价文法。新的文法适合用于 LL(1) 分析等需要无左递归的解析方法。

**最终消除左递归后的文法**：

```
A → Ba A’ | c A’
A’ → a A’ | ε

B → c A’ b B’ | d B’
B’ → b B’ | a A’ b B’ | ε
```
4. 对于一个文法若消除了左递归、提取了左公因子后是否一定为LL(1)文法？
试对下面文法进行改写，并对改写后的文法进行判断。
S → Ab| Ba
A → aA | a
B → a

对于一个文法，消除左递归和提取左公因子虽然是将文法转换为LL(1)文法的必要步骤，但并不能保证最终得到的文法一定是LL(1)的。这是因为即使进行了这些转换，文法中可能仍存在多个产生式具有相同的起始符号，导致它们的FIRST集之间存在交集，从而违反了LL(1)文法的条件。

### 改写文法

给定的文法为：

```
S → Ab | Ba
A → aA | a
B → a
```

根据要求，仅对A的产生式进行左公因子提取。

1. **提取A的左公因子：**

   原产生式：
   ```
   A → aA | a
   ```

   提取左公因子“a”：
   ```
   A → aA'
   A' → A | ε
   ```

2. **改写后的文法：**

   ```
   S → Ab | Ba
   A → aA'
   A' → aA' | ε
   B → a
   ```

### LL(1)分析

进行LL(1)分析需要计算每个非终结符的FIRST集和FOLLOW集，并检查是否满足LL(1)的条件。

1. **计算FIRST集：**

   - **FIRST(S):**
     ```
     S → Ab | Ba
     FIRST(Ab) = FIRST(A) = {a}
     FIRST(Ba) = FIRST(B) = {a}
     所以，FIRST(S) = {a}
     ```

   - **FIRST(A):**
     ```
     A → aA'
     FIRST(A) = {a}
     ```

   - **FIRST(A'):**
     ```
     A' → aA' | ε
     FIRST(A') = {a, ε}
     ```

   - **FIRST(B):**
     ```
     B → a
     FIRST(B) = {a}
     ```

2. **计算FOLLOW集：**

   - **FOLLOW(S):**
     ```
     S 是开始符号，FOLLOW(S) = {$}
     ```

   - **FOLLOW(A):**
     ```
     从产生式 S → Ab 可以看出，A 后跟的是 b
     所以，FOLLOW(A) = {b}
     ```

   - **FOLLOW(A'):**
     ```
     从产生式 A → aA' 可以看出，A' 后没有符号
     所以，FOLLOW(A') = FOLLOW(A) = {b}
     ```

   - **FOLLOW(B):**
     ```
     从产生式 S → Ba 可以看出，B 后跟的是 a
     所以，FOLLOW(B) = {a}
     ```

3. **检查LL(1)条件：**

   对于每个非终结符，检查其产生式的FIRST集是否互不相交，并且如果某个产生式可以推导出ε，则其FIRST集与FOLLOW集也应互不相交。

   - **对于S：**
     ```
     产生式 S → Ab 和 S → Ba
     FIRST(Ab) = {a}
     FIRST(Ba) = {a}
     两个产生式的FIRST集有交集 {a}
     ```
     因此，在遇到输入符号a时，无法确定选择哪个产生式，这违反了LL(1)的条件。

### 结论

通过仅对A的产生式进行左公因子提取，改写后的文法仍然不是LL(1)文法。要使其成为LL(1)文法，可能需要进一步的转换，如重写产生式或引入更多的非终结符，以消除FIRST集的冲突。
