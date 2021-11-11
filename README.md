# Antlr4 Lexer

{% code title="CommonLexerRules.g4" %}
```antlr4
lexer grammar CommonLexerRules;

ADD : '+' ;
SUB : '-' ;
MUL : '*' ;
DIV : '/' ;

ID : [a-zA-Z]+ ;
INT : [0-9]+ ;
WS : [ \r\n\t]+ -> skip ;
```
{% endcode %}

```
antlr4 CommonLexerRules.g4
```

```
grun CommonLexerRules tokens -tokens
```

