本文同步发表于[我的 github 博客](https://litjohn.github.io/posts/start-to-build-a-compiler/)。

光剑的后日谈 #1. 在文艺复兴之后，FP 也要大胜利！

在[这篇文章](https://www.luogu.com.cn/article/ha4xtgpc)的讨论区中，我进行了引流。为了报答作者，我写了这篇文章。

上面那篇文章中使用了一种伪代码语法来表示 lambda calculus。不过这种语法并不能直接运行。怎么办呢？让我们来构建一个源到源编译器！

要写一种语言的编译器，显然我们需要知道它的语法和词法。这是第一步。另外还有语义，但是它是 lambda 的一种表示法，所以语义已经被 lambda 定义了，无需考虑。

首先是词法。这种语言会出现哪些 tokens？

分类一下。lambda 的核心是函数抽象和函数应用。函数抽象用到哪些 tokens？

- func 关键字。
- 冒号。
- return 关键字。
- 参数列表和括号。

函数应用？

- 括号

另外还有一个扩展语法，绑定创建。

- 等号（赋值运算符）

然后我们需要定义它的语法。这篇文章使用的是单参数的原始 lambda，方便了我们的实现。

```plaintext
identifier ::= symbol
abstract-exp ::= func : (identifier) { return val-exp }
apply-exp ::= val-exp(val-exp)
bind-exp ::= identifier = val-exp
val-exp ::= identifier | abstract-exp | apply-exp
lc-exp ::= val-exp | bind-exp
```

看来挺简单的。

在定义了词法和语法之后，就是解析环节了。我们需要把字符串解析成 token 流，从而需要一个 lexer。

等于号，冒号和括号是可用的分隔符。

```racket
#lang racket

(define (get-a-token str)
    (define res '())
    (let loop ([i 0])
        (if (< i (string-length str))
            (case (string-ref str i)
                [(#\space) 
                 (if (null? res) 
                    (loop (+ i 1))
                    (values #f 
                        (string->symbol (list->string (reverse res)))
                        (substring str (+ i 1) (string-length str))))]

                [(#\: #\( #\) #\= #\{ #\}) 
                 (values 
                    (if (null? res) #f (string->symbol (list->string (reverse res))))
                    (string->symbol (string (string-ref str i)))
                    (substring str (+ i 1) (string-length str)))]

                [else (set! res (cons (string-ref str i) res)) (loop (+ i 1))])

            (values
                (string->symbol (list->string (reverse res)))
                #f
                ""))))

(define (lexer str)
    (if (= (string-length str) 0)
        '()
        (let-values ([(s1 s2 rest) (get-a-token str)])
            (cond
                [(and s1 s2) (cons s1 (cons s2 (lexer rest)))]
                [s1 (list s1)]
                [s2 (cons s2 (lexer rest))]
                [else (error "There's something wrong with lexer!")]))))
```

写的非常唐，不过能用就行了。存在一个问题是换行符，但是这里又有 CRLF，CR 和 LF 的问题，就很难处理。索性不处理了，压行算了。

这里还有另一个更严重的问题：我们调用了很多 substring，这导致复杂度退化成平方了。如何解决呢？

其实很简单，我们的扫描步骤是顺序访问的，那么使用 racket 的经典数据结构列表就可以了。下面是重写之后的 lexer：

```racket
#lang racket
;;; ======================================================
;;;  新版 Lexer (基于列表处理的函数式风格)
;;; ======================================================

;; 辅助函数: 判断一个字符是否为分隔符
(define (delimiter? c)
  (member c '(#\: #\( #\) #\= #\{ #\})))

;; 辅助函数: 判断一个字符是否为空白符
(define (whitespace? c)
  (char-whitespace? c))

;; 核心辅助函数: 从字符列表开头读取一个完整的单词(标识符或关键字)
;; 返回两个值: 1. 单词转换成的符号  2. 剩余的字符列表
(define (read-word char-list)
  (let loop ([cs char-list] [acc '()])
    (if (or (null? cs)
            (let ([c (car cs)])
              (or (whitespace? c) (delimiter? c))))
        ;; 遇到空白或分隔符，或者列表为空，结束读取
        (values (string->symbol (list->string (reverse acc)))
                cs)
        ;; 否则，将当前字符加入累加器，继续处理剩余列表
        (loop (cdr cs) (cons (car cs) acc)))))

;; 主词法分析循环
(define (tokenize-loop char-list)
  (if (null? char-list)
      '() ; 输入列表为空，返回空列表
      (let ([c (car char-list)])
        (cond
          ;; 1. 如果是空白符，则忽略它，直接处理剩余部分
          [(whitespace? c)
           (tokenize-loop (cdr char-list))]

          ;; 2. 如果是单个字符的分隔符，将其作为 token，然后处理剩余部分
          [(delimiter? c)
           (cons (string->symbol (string c))
                 (tokenize-loop (cdr char-list)))]

          ;; 3. 否则，它是一个单词的开始，调用 read-word 来读取它
          [else
           (let-values ([(word remaining-chars) (read-word char-list)])
             (cons word (tokenize-loop remaining-chars)))]))))

;; Lexer 的公共接口
;; 它将字符串转换为字符列表，然后启动主循环
(define (lexer str)
  (tokenize-loop (string->list str)))
```

下一步是可爱的 parser。把 token 流解析成 AST。这一步用 EOPL 模块的 define-datatype 会很好，但是我不想用。于是使用 racket 自己的 struct 作为 AST 节点。

另外在这一步我用了一些不是非常 lisp-style 的做法，具体而言把符号列表的 token 流转化为 vector 来操作，用下标区间来标定当前在解析哪部分。

```racket
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;; 1. AST Node Definitions
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
(struct identifier (name) #:transparent)
(struct abstract-exp (param body) #:transparent)
(struct apply-exp (rator rand) #:transparent)
(struct bind-exp (name val) #:transparent)

(define (is-keyword? s)
  (member s '(func : ( ) { } return =)))

(define (is-identifier-token? t)
  (and (symbol? t) (not (is-keyword? t))))


;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;; 2. Parser
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;

(define (parse tokens)
  (let-values ([(ast next-i) (parse-lc-exp (list->vector tokens) 0)])
    (if (= next-i (vector-length (list->vector tokens)))
        ast
        (error "Parsing failed: unexpected tokens remain at the end." (vector-ref (list->vector tokens) next-i)))))

(define (parse-lc-exp tokens i)
  (let ([tok1 (vector-ref tokens i)])
    (if (and (is-identifier-token? tok1)
             (< (+ i 1) (vector-length tokens))
             (eq? (vector-ref tokens (+ i 1)) '=))
        (parse-bind-exp tokens i)
        (parse-val-exp tokens i))))

(define (parse-bind-exp tokens i)
  (let* ([id-name (vector-ref tokens i)]
         [val-start-i (+ i 2)])
    (let-values ([(val-ast next-i) (parse-val-exp tokens val-start-i)])
      (values (bind-exp (identifier id-name) val-ast) next-i))))

; 使用 |(|, |)|, |{|, |}| 以防止与读取器冲突
(define (parse-abstract-exp tokens i)
  (let* ([param-name (vector-ref tokens (+ i 3))]
         [body-start-i (+ i 7)])
    (unless (and (eq? (vector-ref tokens i) 'func)
                 (eq? (vector-ref tokens (+ i 1)) ':)
                 (eq? (vector-ref tokens (+ i 2)) '|(|)
                 (is-identifier-token? param-name)
                 (eq? (vector-ref tokens (+ i 4)) '|)|)
                 (eq? (vector-ref tokens (+ i 5)) '|{|)
                 (eq? (vector-ref tokens (+ i 6)) 'return))
      (error "Invalid function definition syntax"))
    (let-values ([(body-ast body-end-i) (parse-val-exp tokens body-start-i)])
      (unless (and (< body-end-i (vector-length tokens))
                   (eq? (vector-ref tokens body-end-i) '|}|))
        (error "Expected '}' at the end of function body"))
      (values (abstract-exp (identifier param-name) body-ast)
              (+ body-end-i 1)))))

(define (parse-val-exp tokens i)
  (let-values ([(left-ast next-i)
                (let ([tok (vector-ref tokens i)])
                  (cond
                    [(eq? tok 'func) (parse-abstract-exp tokens i)]
                    [(is-identifier-token? tok) (values (identifier tok) (+ i 1))]
                    [else (error "Invalid value expression: expected identifier or function at index" i)]))])
    (let loop ([current-ast left-ast] [current-i next-i])
      ; 检查是否是函数应用
      (if (and (< current-i (vector-length tokens))
               (eq? (vector-ref tokens current-i) '|(|))
        ; 是: 解析参数, 构建 apply-exp, 然后继续循环
        (let-values ([(arg-ast arg-end-i) (parse-val-exp tokens (+ current-i 1))])
          (unless (and (< arg-end-i (vector-length tokens))
                       (eq? (vector-ref tokens arg-end-i) '|)|))
            (error "Expected ')' to close function application"))
          (loop (apply-exp current-ast arg-ast) (+ arg-end-i 1)))
        ; 否: 循环结束, 返回当前积累的 AST 和索引
        (values current-ast current-i)))))
```

看着有点令人头大。其实这是 AIGC 代码，感谢 Gemini 2.5 Pro 给予的帮助。

parser 写起来确实令人无比头大。各种神秘的位置编码和 hack……我开始怀疑自己为什么不使用（或者不打算使用）match 来进行分发。这段代码让人完全没有调试的想法。

这些解析大致使用了递归下降法。这是一种非常经典的方法，具体而言，我们一开始以归纳（inductive）的形式定义了语言的语法。然后我们将具体语法转换为抽象语法，为每种语法元素定义一种节点类型，为每种节点类型定义一个过程进行解析，解析的过程就变成了这些函数互相递归的过程。

也即是一种数据驱动的编程：一个语法元素对应一种 AST 节点和一个 struct 定义，解析它对应一个过程，元素可能的多种情形对应 cond 的不同分支，归纳的定义对应函数间的相互递归。

要进行更多的了解，推荐阅读《essential of programming languages》（即 EOPL）。

进行一些测试：

```racket
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;; 3. Testing
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;

(define test-str-1 "id = func : (x) { return x }")
(define test-str-2 "apply = func : (f) { return func : (x) { return f(x) } }")
(define test-str-3 "id(y)")
(define test-str-4 "get-y(f)(x)")

(displayln "--- Test 1: Binding ---")
(pretty-print (parse (lexer test-str-1)))

(displayln "--- Test 2: Currying ---")
(pretty-print (parse (lexer test-str-2)))

(displayln "--- Test 3: Simple Application ---")
(pretty-print (parse (lexer test-str-3)))

(displayln "--- Test 4: Chained Application ---")
(pretty-print (parse (lexer test-str-4)))
```

结果：

```plaintext
--- Test 1: Binding ---
(bind-exp (identifier 'id) (abstract-exp (identifier 'x) (identifier 'x)))
--- Test 2: Currying ---
(bind-exp
 (identifier 'apply)
 (abstract-exp
  (identifier 'f)
  (abstract-exp (identifier 'x) (apply-exp (identifier 'f) (identifier 'x)))))
--- Test 3: Simple Application ---
(apply-exp (identifier 'id) (identifier 'y))
--- Test 4: Chained Application ---
(apply-exp (apply-exp (identifier 'get-y) (identifier 'f)) (identifier 'x))
```

可以看到，parse 的结果是正确的。

在建出 AST 之后，进行语法转换就非常容易了。因为源语言和目标语言只不过是 lc-exp 的两种不同表示法（具体语法，concrete syntax）。有了抽象语法，转换为不同的具体语法易如反掌。

吸取了 parser 的惨痛教训，这次我们使用 match 进行操作。

```racket
(define (convert node)
    (match node
        [(bind-exp name val) `(define ,(convert name) ,(convert val))]
        [(apply-exp rator rand) `(,(convert rator) ,(convert rand))]
        [(abstract-exp arg body) `(lambda (,(convert arg)) ,(convert body))]
        [(identifier id) id]))
```

然后一些测试。我们用上次的老例子：

```racket
(define test-str-1 "id = func : (x) { return x }")
(define test-str-2 "apply = func : (f) { return func : (x) { return f(x) } }")
(define test-str-3 "id(y)")
(define test-str-4 "get-y(f)(x)")

(displayln "--- Test 1: Binding ---")
(displayln (convert (parse (lexer test-str-1))))

(displayln "--- Test 2: Currying ---")
(displayln (convert (parse (lexer test-str-2))))

(displayln "--- Test 3: Simple Application ---")
(displayln (convert (parse (lexer test-str-3))))

(displayln "--- Test 4: Chained Application ---")
(displayln (convert (parse (lexer test-str-4))))
```

结果：

```plaintext
--- Test 1: Binding ---
(define id (lambda (x) x))
--- Test 2: Currying ---
(define apply (lambda (f) (lambda (x) (f x))))
--- Test 3: Simple Application ---
(id y)
--- Test 4: Chained Application ---
((get-y f) x)
```

成功了！

---

这样，我们就可以将伪代码直接送进 chez scheme 或者 racket 运行了！

更重要的是，我们亲手走了一遍扫描（词法分析）、解析（语法分析）和代码生成的编译流程。构建了一个简易的源到源编译器。

如果想要了解更多，推荐阅读 EOPL（《essential of programming languages》），friedman 教授的著作。其中第一章就是归纳式的语法，全书会带领你实现各种各样语言的解释器以学习种种语言特性。

最后是完整代码：

```racket
#lang racket
;;; ======================================================
;;;  新版 Lexer (基于列表处理的函数式风格)
;;; ======================================================

;; 辅助函数: 判断一个字符是否为分隔符
(define (delimiter? c)
  (member c '(#\: #\( #\) #\= #\{ #\})))

;; 辅助函数: 判断一个字符是否为空白符
(define (whitespace? c)
  (char-whitespace? c))

;; 核心辅助函数: 从字符列表开头读取一个完整的单词(标识符或关键字)
;; 返回两个值: 1. 单词转换成的符号  2. 剩余的字符列表
(define (read-word char-list)
  (let loop ([cs char-list] [acc '()])
    (if (or (null? cs)
            (let ([c (car cs)])
              (or (whitespace? c) (delimiter? c))))
        ;; 遇到空白或分隔符，或者列表为空，结束读取
        (values (string->symbol (list->string (reverse acc)))
                cs)
        ;; 否则，将当前字符加入累加器，继续处理剩余列表
        (loop (cdr cs) (cons (car cs) acc)))))

;; 主词法分析循环
(define (tokenize-loop char-list)
  (if (null? char-list)
      '() ; 输入列表为空，返回空列表
      (let ([c (car char-list)])
        (cond
          ;; 1. 如果是空白符，则忽略它，直接处理剩余部分
          [(whitespace? c)
           (tokenize-loop (cdr char-list))]

          ;; 2. 如果是单个字符的分隔符，将其作为 token，然后处理剩余部分
          [(delimiter? c)
           (cons (string->symbol (string c))
                 (tokenize-loop (cdr char-list)))]

          ;; 3. 否则，它是一个单词的开始，调用 read-word 来读取它
          [else
           (let-values ([(word remaining-chars) (read-word char-list)])
             (cons word (tokenize-loop remaining-chars)))]))))

;; Lexer 的公共接口
;; 它将字符串转换为字符列表，然后启动主循环
(define (lexer str)
  (tokenize-loop (string->list str)))

;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;; 1. AST Node Definitions (保持不变)
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
(struct identifier (name) #:transparent)
(struct abstract-exp (param body) #:transparent)
(struct apply-exp (rator rand) #:transparent)
(struct bind-exp (name val) #:transparent)

(define (is-keyword? s)
  (member s '(func : ( ) { } return =)))

(define (is-identifier-token? t)
  (and (symbol? t) (not (is-keyword? t))))


;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;; 2. Parser (修正版)
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;

(define (parse tokens)
  (let-values ([(ast next-i) (parse-lc-exp (list->vector tokens) 0)])
    (if (= next-i (vector-length (list->vector tokens)))
        ast
        (error "Parsing failed: unexpected tokens remain at the end." (vector-ref (list->vector tokens) next-i)))))

(define (parse-lc-exp tokens i)
  (let ([tok1 (vector-ref tokens i)])
    (if (and (is-identifier-token? tok1)
             (< (+ i 1) (vector-length tokens))
             (eq? (vector-ref tokens (+ i 1)) '=))
        (parse-bind-exp tokens i)
        (parse-val-exp tokens i))))

(define (parse-bind-exp tokens i)
  (let* ([id-name (vector-ref tokens i)]
         [val-start-i (+ i 2)])
    (let-values ([(val-ast next-i) (parse-val-exp tokens val-start-i)])
      (values (bind-exp (identifier id-name) val-ast) next-i))))

; 修正: 使用 |(|, |)|, |{|, |}|
(define (parse-abstract-exp tokens i)
  (let* ([param-name (vector-ref tokens (+ i 3))]
         [body-start-i (+ i 7)])
    (unless (and (eq? (vector-ref tokens i) 'func)
                 (eq? (vector-ref tokens (+ i 1)) ':)
                 (eq? (vector-ref tokens (+ i 2)) '|(|)
                 (is-identifier-token? param-name)
                 (eq? (vector-ref tokens (+ i 4)) '|)|)
                 (eq? (vector-ref tokens (+ i 5)) '|{|)
                 (eq? (vector-ref tokens (+ i 6)) 'return))
      (error "Invalid function definition syntax"))
    (let-values ([(body-ast body-end-i) (parse-val-exp tokens body-start-i)])
      (unless (and (< body-end-i (vector-length tokens))
                   (eq? (vector-ref tokens body-end-i) '|}|))
        (error "Expected '}' at the end of function body"))
      (values (abstract-exp (identifier param-name) body-ast)
              (+ body-end-i 1)))))

; 修正: 修复 loop 的逻辑, 确保有 else 分支, 并使用正确的符号
(define (parse-val-exp tokens i)
  (let-values ([(left-ast next-i)
                (let ([tok (vector-ref tokens i)])
                  (cond
                    [(eq? tok 'func) (parse-abstract-exp tokens i)]
                    [(is-identifier-token? tok) (values (identifier tok) (+ i 1))]
                    [else (error "Invalid value expression: expected identifier or function at index" i)]))])
    (let loop ([current-ast left-ast] [current-i next-i])
      ; 检查是否是函数应用
      (if (and (< current-i (vector-length tokens))
               (eq? (vector-ref tokens current-i) '|(|))
        ; 是: 解析参数, 构建 apply-exp, 然后继续循环
        (let-values ([(arg-ast arg-end-i) (parse-val-exp tokens (+ current-i 1))])
          (unless (and (< arg-end-i (vector-length tokens))
                       (eq? (vector-ref tokens arg-end-i) '|)|))
            (error "Expected ')' to close function application"))
          (loop (apply-exp current-ast arg-ast) (+ arg-end-i 1)))
        ; 否: 循环结束, 返回当前积累的 AST 和索引
        (values current-ast current-i)))))

(define (convert node)
    (match node
        [(bind-exp name val) `(define ,(convert name) ,(convert val))]
        [(apply-exp rator rand) `(,(convert rator) ,(convert rand))]
        [(abstract-exp arg body) `(lambda (,(convert arg)) ,(convert body))]
        [(identifier id) id]))

;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;; 3. Testing
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
#|
(define test-str-1 "id = func : (x) { return x }")
(define test-str-2 "apply = func : (f) { return func : (x) { return f(x) } }")
(define test-str-3 "id(y)")
(define test-str-4 "get-y(f)(x)")

(displayln "--- Test 1: Binding ---")
(displayln (convert (parse (lexer test-str-1))))

(displayln "--- Test 2: Currying ---")
(displayln (convert (parse (lexer test-str-2))))

(displayln "--- Test 3: Simple Application ---")
(displayln (convert (parse (lexer test-str-3))))

(displayln "--- Test 4: Chained Application ---")
(displayln (convert (parse (lexer test-str-4))))
|#
```


