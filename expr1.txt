%{#include<stdio.h>
#include<stdlib.h>
#ifndef YYSTYPE
#define YYSTYPE double  //计算表达式，$$的类型
#endif
int yylex(); //词法分析
extern int yyparse();
FILE* yyin;
void yyerror(const char* s);
%}
//规则段
%token NUMBER //将 yylval 隐式赋值给终结符的属性值
%token ADD //为每个单词定义一种单词类别
%token SUB //用定义名称为 ADD 的 token 表示加号，首先需要声明该 token
%token MUL // *
%token DIV // /
%token LEF // (
%token RIG // )
//声明运算符的左右结合性及优先级
//按照优先级由低到高的顺序声明，同一级的运算符放在同一行。
%left ADD SUB
%left MUL DIV
%right UMINUS // -

%%   //规则段。语法分析。上下文无关文法及翻译模式

lines : lines expr ';'{ printf("%f\n", $2); }//使用分号替换 lines 产生式中的 ‘\n’
      | lines ';'
      |
      ;

expr  : expr ADD expr{ $$ = $1 + $3; } //在规则段涉及加号的位置替换为 ADD
      | expr SUB expr{ $$ = $1 - $3; }
      | expr MUL expr{ $$ = $1 * $3; }
      | expr DIV expr{ $$ = $1 / $3; }
      | LEF expr RIG{$$ = $2; }
      | SUB expr %prec UMINUS{$$ = -$2;}
      | NUMBER{ $$ = $1; }
      ;
%%

// program section

int yylex()
{
 // place your token retrieving code here
    int t;
    while (1) {
        t = getchar();
        switch (t) {
            case ' ':
            case '\t':
            case '\n':
                break;//空格、制表符、回车换行;

            case '0' ... '9': {//将 NUMBER视为终结符，添加语义动作
                yylval = 0;
                while (t>='0'&&t<='9') {
                    yylval = yylval * 10 + t - '0';
                    t = getchar();
                }
                ungetc(t, stdin);//多余字再次放回到缓冲区去
                return NUMBER;
                break;
                }
            case '+':
                return ADD; //在词法分析函数 yylex 写明加号的情况如何处理
                break;
            case '-':
                return SUB;
                break;
            case '*':
                return MUL;
                break;
            case '/':
                return DIV;
                break;
            case '(':
                return LEF;
                break;
            case ')':
                return RIG;
                break;

            default:
                return t;
                break;
    }
  }
}
int main(void)
{
 yyin = stdin;
 do {
  yyparse();
 } while (!feof(yyin));
 return 0;
}
void yyerror(const char* s) {
 fprintf(stderr, "Parse error:%s\n", s);
 exit(1);
}