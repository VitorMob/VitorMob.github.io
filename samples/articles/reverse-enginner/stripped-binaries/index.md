---
layout: article
---

# Stripped Binaries 
<br>

> Author: VitorMob 

> Data: 18/03/2022

---

Nesse artigo irei abordar o tema stripped binaries, de acordo com meus estudos e pesquisas feitos atualmente em relação ao formato [ELF 
(Executable and Linkable Format)](https://en.wikipedia.org/wiki/Executable_and_Linkable_Format), não irei abordar outros formatos. como exemplo 
[PE (Portable Executable)](https://pt.wikipedia.org/wiki/Portable_Executable). Para compreenção deste artigo irei utilizar a [linguagem  C](https://pt.wikipedia.org/wiki/C_(linguagem_de_programa%C3%A7%C3%A3o)) para ilustração de codigos e binários compilados. Irei disponibilizar um livro  no qual tem como foco principal analisar binários ELF e PE, e um curso sobre Programação moderna em C feito pelo mente binaria totalmente gratuito.	

Em relação aos meus estudos, analisar binário para Engenharia Reversa e Analise de Malware, tem com foco principal buscar por referências de funções entre outros symbols para ajudar a entender o disassembly e o fluxo que o código esta tomando, é podemos consegui-las  através do [Symbol Table Section](https://docs.oracle.com/cd/E23824_01/html/819-0690/chapter6-79797.html), que o compilador ira gerar para fornecer informações basicas ao linker, para binários ELF os symbols para debbuger normalmente são gerados através do [DWARF](https://en.wikipedia.org/wiki/DWARF). No caso do PE os symbols são gerados através do [PDB (Microsoft Portable Debugging)](https://learn.microsoft.com/pt-br/visualstudio/debugger/specify-symbol-dot-pdb-and-source-files-in-the-visual-studio-debugger?view=vs-2022), geralmente os symbols para debbuger em relação ao DWARF é inserido no binário, enquanto que o formato PDB é tipicamente gerado em um arquivo separado. é por esse motivo não irei citar o formato PE novamente neste artigo.

Vamos partir para uma demonstração de um código basico escrito na linguagem C, que tem como foco, um simples `Hello, World`:

``` compiler : gcc -o main main.c ```

```c
#include <stdio.h> 

int main( void )
{
    puts(“Hello, World”);
    return 0; 
}
```

``` execute : readelf  —syms main  ```

> output: 	


   | Num  |  Value        |    Size | Type| Bind  | Vis   |  Ndx  | Name |
   |:------|:--------------|:--------|:----|:------|:------|:-------|:------| 
   |  0 | 0000000000000000  |   0  | NOTYPE   | LOCAL |   DEFAULT|  UND |    |
   |  1 | 0000000000000000  |   0  | FILE     | LOCAL |   DEFAULT|  ABS |  abi-note.c|
   |  2 | 000000000000039c  |   32 | OBJECT   | LOCAL |   DEFAULT|    4 |  __abi_tag|
   |  3 | 0000000000000000  |   0  | FILE     | LOCAL |   DEFAULT|  ABS |  init.c|
   |  4 | 0000000000000000  |   0  | FILE     | LOCAL |   DEFAULT|  ABS |  crtstuff.c|
   |  5 | 0000000000001070  |   0  | FUNC     | LOCAL |   DEFAULT|   14 |  deregister_tm_clones|
   |  6 | 00000000000010a0  |   0  | FUNC     | LOCAL |   DEFAULT|   14 |  register_tm_clones|
   |  7 | 00000000000010e0  |   0  | FUNC     | LOCAL |   DEFAULT|   14 |  __do_global_dtors_aux|
   |  8 | 0000000000004030  |   1  | OBJECT   | LOCAL |   DEFAULT|   25 |	 completed.0|
   |  9 | 0000000000003df0  |   0  | OBJECT   | LOCAL |   DEFAULT|   20 |	 __do_global_dtor[...]|
   | 10 | 0000000000001130  |   0  | FUNC     | LOCAL |   DEFAULT|   14 |  frame_dummy|
   | 11 | 0000000000003de8  |   0  | OBJECT   | LOCAL |   DEFAULT|   19 |  __frame_dummy_in[...]|
   | 12 | 0000000000000000  |   0  | FILE     | LOCAL |   DEFAULT|  ABS |	 main.c|
   | 13 | 0000000000000000  |   0  | FILE     | LOCAL |   DEFAULT|  ABS |  crtstuff.c|
   | 14 | 00000000000020b0  |   0  | OBJECT   | LOCAL |   DEFAULT|   18 |  __FRAME_END__|
   | 15 | 0000000000000000  |   0  | FILE     | LOCAL |   DEFAULT|  ABS | |
   | 16 | 0000000000003df8  |   0  | OBJECT   | LOCAL |   DEFAULT|   21 |	 _DYNAMIC|
   | 17 | 0000000000002014  |   0  | NOTYPE   | LOCAL |   DEFAULT|   17 |  __GNU_EH_FRAME_HDR|
   | 18 | 0000000000004000  |   0  | OBJECT   | LOCAL |   DEFAULT|   23 |  _GLOBAL_OFFSET_TABLE_|
   | 19 | 0000000000000000  |   0  | FUNC     | GLOBAL |  DEFAULT|  UND |  __libc_start_mai[...]|
   | 20 | 0000000000000000  |   0  | NOTYPE   | WEAK |    DEFAULT|  UND |  _ITM_deregisterT[...]|
   | 21 | 0000000000004020  |   0  | NOTYPE   | WEAK |    DEFAULT|   24 |  data_start|
   | 22 | 0000000000000000  |   0  | FUNC     | GLOBAL |  DEFAULT|  UND |  puts@GLIBC_2.2.5|
   | 23 | 0000000000004030  |   0  | NOTYPE   | GLOBAL |  DEFAULT|   24 |  _edata|
   | 24 | 0000000000001154  |   0  | FUNC     | GLOBAL |  HIDDEN|    15 |  _fini|
   | 25 | 0000000000004020  |   0  | NOTYPE   | GLOBAL |  DEFAULT|   24 |  __data_start|
   | 26 | 0000000000000000  |   0  | NOTYPE   | WEAK |    DEFAULT|  UND |  __gmon_start__|
   | 27 | 0000000000004028  |   0  | OBJECT   | GLOBAL |  HIDDEN|    24 |  __dso_handle|
   | 28 | 0000000000002000  |   4  | OBJECT   | GLOBAL |  DEFAULT|   16 |  _IO_stdin_used|
   | 29 | 0000000000004038  |   0  | NOTYPE   | GLOBAL |  DEFAULT|   25 |	 _end|
   | 30 | 0000000000001040  |   38 | FUNC    | GLOBAL |  DEFAULT|   14 | 	 _start|
   | 31 | 0000000000004030  |   0  | NOTYPE   | GLOBAL |  DEFAULT|   25 |  __bss_start|
   | 32 | 0000000000001139  |   26 | FUNC    | GLOBAL |  DEFAULT|   14 |	 main|
   | 33 | 0000000000004030  |   0  | OBJECT   | GLOBAL |  HIDDEN|    24 |  __TMC_END__|
   | 34 | 0000000000000000  |   0  | NOTYPE   | WEAK |    DEFAULT|  UND |	 _ITM_registerTMC[...]|
   | 35 | 0000000000000000  |   0  | FUNC     | WEAK |    DEFAULT|  UND |	 __cxa_finalize@G[...]|
   | 36 | 0000000000001000  |   0  | FUNC     | GLOBAL |  HIDDEN|    12 |	 _init|


> Obs: a flag  `—syms` ira trazer apenas a tabela de symbols do nosso binário. No caso a `.symtab`. 


Utilizamos uma tool [readelf](https://man7.org/linux/man-pages/man1/readelf.1.html) bastante util para parser de ELF, podemos obter variadas informações em relação ao binário, 
 `Section Headers, Program Header …` porém esse não é o nosso foco no momento. apenas obtemos as informações da nossa section `.symtab`. No canto esquerdo chamado `Value` podemos 
visualizar o offset, em seguida ele nós traz a informação do que se trata cada symbol, podemos notar o symbol **(32)** offset `0000000000001139` Name `main` com o Type `FUNC` 
ou seja este symbol se trata de uma função, percebe-se que ele entraga o tamanho  da nossa main no campo `Size` contendo `26 bytes`, preste bastante atenção, percebe-se que o offset do nosso symbol **(22)** Name `puts@GLIBC_2.2.5` ainda não foi resolvido, porque o linker precisa resolver em load time, isso ocorre porque o nosso binário esta linkado dinamicamente com a biblioteca, não irei entrar em detalhes em relação a linkagem. 

> Recomendo fazer proveito do [manpages](https://man7.org/linux/man-pages/man5/elf.5.html) no tópico `elf` para verificar o que cada symbol representa no nosso binário. 

Por padrão o [gcc](https://man7.org/linux/man-pages/man1/gcc.1.html) faz a chamada para o linker [ld](https://linux.die.net/man/1/ld), é entrega o binário perfeito para ser executado, porém ele não faz a chamada do [strip](https://man7.org/linux/man-pages/man1/strip.1.html) no qual tende a remover os symbols do nosso binário, podemos verificar o tamanho do nosso binário em bytes usando o [du](https://man7.org/linux/man-pages/man1/du.1.html) um utilitário linux para estimar o uso do espaço no arquivo.

``` execute : du -b main ```
> output : 

```
15936  main
```
> Obs: a flag `-b` irá trazer a contagem em bytes, équivale para `--apparent-size --block-size=1`.

Podemos notar que o nosso binário executavel esta com ```15936b```, lembrando, o nosso binário, não esta stripped ou seja, todos os symbols mostrado acima, esta no nosso binário, acarretando assim em um binário mais robusto, podemos obter informações bastante interessantes em relação a nossa section `.symtab`. Vamos dar uma conferida nas `relocs` que informa ao linker para resolver as referencias tanto para chamadas de funções ou ponteiros para string. Vamos conferir utilizando novamente o nosso `readelf`.

``` execute: readelf —relocs main ```

> output: 


|  Offset |  Info |   Type  |  Sym. Value | Sym. Name + Addend | 
|:--------|:------|:--------|:------------|:-------------------|
|000000004018 | 000300000007 | R_X86_64_JUMP_SLO | 0000000000000000 | puts@GLIBC_2.2.5 + 0 | 


Iremos obter a nossa chamada para a `puts@GLIBC`.

> **Offset** : `000000004018` É o descolamento onde o valor do symbol deve ir.

> **Info** : `000300000007` O tipo (termina o cálculo exato depende do arch) e o índice do símbolo na symtab.

> **Type** : `R_X86_64_JUMP_SLO` Tipo do símbolo de acordo com a ABI.

> **Sym. Value** :  `0000000000000000` O valor Sym é o adendo a ser adicionado à resolução do símbolo 

> **Sym. Name + Addend** : `puts@GLIBC_2.2.5 + 0` Nome do Sym e adendo - uma impressão bonita do nome do símbolo + adendo.


Vamos dar uma olhada no disassembly usando o utilitário [objdump](https://man7.org/linux/man-pages/man1/objdump.1.html).

``` execute: objdump -M intel -d main ```

> output:

```as
Disassembly of section .init:

0000000000001000 <_init>:
    1000:	f3 0f 1e fa          	    endbr64 
    1004:	48 83 ec 08          	    sub    rsp,0x8
    1008:	48 8b 05 d9 2f 00 00 	    mov    rax,QWORD PTR [rip+0x2fd9]        # 3fe8 <__gmon_start__@Base>
    100f:	48 85 c0                         test   rax,rax
    1012:	74 02                            je     1016 <_init+0x16>
    1014:	ff d0                            call   rax
    1016:	48 83 c4 08          	    add    rsp,0x8
    101a:	c3                               ret    

Disassembly of section .plt:

0000000000001020 <puts@plt-0x10>:
    1020:	ff 35 e2 2f 00 00    	    push   QWORD PTR [rip+0x2fe2]        # 4008 <_GLOBAL_OFFSET_TABLE_+0x8>
    1026:	ff 25 e4 2f 00 00    	    jmp    QWORD PTR [rip+0x2fe4]        # 4010 <_GLOBAL_OFFSET_TABLE_+0x10>
    102c:	0f 1f 40 00          	    nop    DWORD PTR [rax+0x0]

0000000000001030 <puts@plt>:
(3) 1030:	ff 25 e2 2f 00 00    	    jmp    QWORD PTR [rip+0x2fe2]        # 4018 <puts@GLIBC_2.2.5>
    1036:	68 00 00 00 00       	    push   0x0
    103b:	e9 e0 ff ff ff       	    jmp    1020 <_init+0x20>

(1) Disassembly of section .text:

0000000000001040 <_start>:
    1040:	f3 0f 1e fa          		endbr64 
    1044:	31 ed                		xor    ebp,ebp
    1046:	49 89 d1             		mov    r9,rdx
    1049:	5e                   		pop    rsi
    104a:	48 89 e2             		mov    rdx,rsp
    104d:	48 83 e4 f0          		and    rsp,0xfffffffffffffff0
    1051:	50                   		push   rax
    1052:	54                   		push   rsp
    1053:	45 31 c0             		xor    r8d,r8d
    1056:	31 c9                		xor    ecx,ecx
    1058:	48 8d 3d da 00 00 00 	          lea    rdi,[rip+0xda]         # 1139 <main>
    105f:	ff 15 73 2f 00 00    	          call   QWORD PTR [rip+0x2f73] # 3fd8 <__libc_start_main@GLIBC_2.34>
    1065:	f4                   		hlt    
    1066:	66 2e 0f 1f 84 00 00 	          cs nop WORD PTR [rax+rax*1+0x0]
    106d:	00 00 00 

...

(2) 0000000000001139 <main>:
    1139:	55                   		push   rbp
    113a:	48 89 e5             		mov    rbp,rsp
    113d:	48 8d 05 c0 0e 00 00 	          lea    rax,[rip+0xec0]        # 2004 <_IO_stdin_used+0x4>
    1144:	48 89 c7             		mov    rdi,rax
    1147:	e8 e4 fe ff ff       		call   1030 <puts@plt>
    114c:	b8 00 00 00 00       	          mov    eax,0x0
    1151:	5d                   		pop    rbp
    1152:	c3                   		ret    
```

> Obs: as flag `-d` ira trazer o disassembly do nosso binário, a flag `–M` especifica qual alvo o disassembly deve formatar, neste caso intel.

Ele nós traz algumas outras sections que eu não irei entrar em detalhes aqui, porém a section **(1)**`.text` responsavel por conter o nosso codigo executavel, localiza-se no  offset `105f` uma chamada a função que ficará responsavel por inicializar a nossa função **(2)**`main`. O disassembly gerado identificou qual symbol se trata a chamada `__libc_start_main@GLIBC_2.34`.

```c
int __libc_start_main(int *(main) (int, char * *, char * *), 
                      int argc, char * * ubp_av, 
                      void (*init) (void), 
                      void (*fini) (void), 
                      void (*rtld_fini) (void), 
                      void (* stack_end));
```
As principais funções da `__libc_start_main` é : 

>  Chama os construtores e destruidores do nosso programa

>  Chama main() com os argumentos corretos

>  Chama exit() com o valor de retorno de main.

>  Verifique se o uid efetivo == uid real (por motivos de segurança)


Então ja temos noção que a `main` é passada por parametro para a função  `__libc_start_main`, podemos notar o offset da nossa `main` sendo passada como parametro, no offset `1058`. Vamos para o offset `0000000000001139` onde se encontra a nossa `main`, visualizando a nossa `main` podemos notar que no offset `1147`, faz uma chamada para o nosso `<puts@plt>`, perceba o disassembly ```call 1030 ```, o linker refere-se a chamada para o puts na section **(3)**`.plt`.

> Uma observação, antes da nossa função `main` ser chamada, a função `_start` gerada pelo gcc ira ser executada primeiro, que ira ser responsavel ​​por tarefas como configurar os argumentos da linha de comando e o ambiente em tempo de execução (__runtime__). 

Como você ja notou, tivemos varias informações sobre o binário, porque os symbols nos ajudou bastante a entender o fluxo, obtemos informações importantes sobre as nossas funções, chamadas de função etc...  vamos agora remover os symbols do nosso binário, usando a ferramenta `strip`, em seguida vamos dar uma olhada no tamanho do nosso binário:

``` execute: strip --strip-all main ; du –b main ```


> output:  


```14408	main ```

> Obs: a flag `--strip-all` é responsavel por remover todos os symbols "desnecessarios" do nosso binario.

`15936b - 14408b = 1528b`, O nosso binário atualmente esta com  `14408b` de tamanho, devido a remoção dos symbols nosso binário “perdeu” `1528b`. Vamos dar uma olhada nos symbols do nosso binário ...

``` execute: readelf --syms main ``` 

> output:


| Num  |   Value  |        Size | Type|     Bind |  Vis|      Ndx Name |
|:-------- |:-------- |:-------- |:--------|:-------- |:-------- |:--------|
|  0| 0000000000000000 | 0 |NOTYPE | LOCAL | DEFAULT | UND | |
|  1| 0000000000000000 | 0 |FUNC   | GLOBAL| DEFAULT | UND | _[...]@GLIBC_2.34 (2)|
|  2| 0000000000000000 | 0 |NOTYPE | WEAK  | DEFAULT | UND | _ITM_deregisterT[...]|
|  3| 0000000000000000 | 0 |FUNC   | GLOBAL| DEFAULT | UND | puts@GLIBC_2.2.5 (3)|
|  4| 0000000000000000 | 0 |NOTYPE | WEAK  | DEFAULT | UND | __gmon_start__|
|  5| 0000000000000000 | 0 |NOTYPE | WEAK  | DEFAULT | UND | _ITM_registerTMC[...]|
|  6| 0000000000000000 | 0 |FUNC   | WEAK  | DEFAULT | UND | [...]@GLIBC_2.2.5 (3)|

Percebeu que a nossa  section `.symtab`, esta vazia, ele não conseguiu fazer o output, pós removemos os symbols... Porém a nossa section `.dynsym` continua com os symbols, por quẽ ? isso ocorre porque o linker precisa resolver em tempo de carregamento (__load time__) as chamadas para funções, por conta do nosso binário esta dinamicamente linkado com a biblioteca, vamos dar uma olhada no disassembly.
	
```as 
Disassembly of section .init:

0000000000001000 <.init>:
    1000:	f3 0f 1e fa          	          endbr64 
    1004:	48 83 ec 08          		sub    rsp,0x8
    1008:	48 8b 05 d9 2f 00 00 		mov    rax,QWORD PTR [rip+0x2fd9]        # 3fe8 <puts@plt+0x2fb8>
    100f:	48 85 c0         			test   rax,rax
    1012:	74 02          		      	je     1016 <puts@plt-0x1a>
    1014:	ff d0                		call   rax
    1016:	48 83 c4 08          		add    rsp,0x8
    101a:	c3                   		ret    

Disassembly of section .plt:

0000000000001020 <puts@plt-0x10>:
    1020:	ff 35 e2 2f 00 00    		    push   QWORD PTR [rip+0x2fe2]        # 4008 <puts@plt+0x2fd8>
    1026:	ff 25 e4 2f 00 00    		    jmp    QWORD PTR [rip+0x2fe4]        # 4010 <puts@plt+0x2fe0>
    102c:	0f 1f 40 00          		    nop    DWORD PTR [rax+0x0]

0000000000001030 <puts@plt>:
    1030:	ff 25 e2 2f 00 00    		    jmp    QWORD PTR [rip+0x2fe2]        # 4018 <puts@plt+0x2fe8>
    1036:	68 00 00 00 00       		    push   0x0
    103b:	e9 e0 ff ff ff       		    jmp    1020 <puts@plt-0x10>

Disassembly of section .text:

0000000000001040 <.text>:
    1040:	f3 0f 1e fa          			endbr64 
    1044:	31 ed                			xor    ebp,ebp
    1046:	49 89 d1             			mov    r9,rdx
    1049:	5e                   			pop    rsi
    104a:	48 89 e2             			mov    rdx,rsp
    104d:	48 83 e4 f0         		 	and    rsp,0xfffffffffffffff0
    1051:	50                   			push   rax
    1052:	54                   			push   rsp
    1053:	45 31 c0            		 	xor    r8d,r8d
    1056:	31 c9                			xor    ecx,ecx
    1058:	48 8d 3d da 00 00 00 		          lea    rdi,[rip+0xda]        # 1139 <puts@plt+0x109>
    105f:	ff 15 73 2f 00 00    		          call   QWORD PTR [rip+0x2f73]        # 3fd8 <puts@plt+0x2fa8>
    1065:	f4                   			hlt    	
    1066:	66 2e 0f 1f 84 00 00 		          cs nop WORD PTR [rax+rax*1+0x0]
    106d:	00 00 00 
    1070:	48 8d 3d b9 2f 00 00 		          lea    rdi,[rip+0x2fb9]        # 4030 <puts@plt+0x3000>
    1077:	48 8d 05 b2 2f 00 00 		          lea    rax,[rip+0x2fb2]        # 4030 <puts@plt+0x3000>
    107e:	48 39 f8            		 	cmp    rax,rdi
    1081:	74 15                			je     1098 <puts@plt+0x68>
    1083:	48 8b 05 56 2f 00 00 		          mov    rax,QWORD PTR [rip+0x2f56]        # 3fe0 <puts@plt+0x2fb0>
    108a:	48 85 c0          		   	          test   rax,rax
    108d:	74 09                			je     1098 <puts@plt+0x68>
    108f:	ff e0                			jmp    rax
    1091:	0f 1f 80 00 00 00 00 		          nop    DWORD PTR [rax+0x0]
    1098:	c3                   			ret    
    1099:	0f 1f 80 00 00 00 00 		          nop    DWORD PTR [rax+0x0]
    10a0:	48 8d 3d 89 2f 00 00 		          lea    rdi,[rip+0x2f89]        # 4030 <puts@plt+0x3000>
    10a7:	48 8d 35 82 2f 00 00 		          lea    rsi,[rip+0x2f82]        # 4030 <puts@plt+0x3000>
    10ae:	48 29 fe             			sub    rsi,rdi
    10b1:	48 89 f0             			mov    rax,rsi
    10b4:	48 c1 ee 3f         		 	shr    rsi,0x3f
    10b8:	48 c1 f8 03         		 	sar    rax,0x3
    10bc:	48 01 c6             			add    rsi,rax
    10bf:	48 d1 fe             			sar    rsi,1
    10c2:	74 14                			je     10d8 <puts@plt+0xa8>
    10c4:	48 8b 05 25 2f 00 00 		          mov    rax,QWORD PTR [rip+0x2f25]        # 3ff0 <puts@plt+0x2fc0>
    10cb:	48 85 c0             			test   rax,rax
    10ce:	74 08                			je     10d8 <puts@plt+0xa8>
    10d0:	ff e0                			jmp    rax
    10d2:	66 0f 1f 44 00 00    		          nop    WORD PTR [rax+rax*1+0x0]
    10d8:	c3                   			ret    	
    10d9:	0f 1f 80 00 00 00 00 	    	          nop    DWORD PTR [rax+0x0]
    10e0:	f3 0f 1e fa          			endbr64 
    10e4:	80 3d 45 2f 00 00 00    		          cmp    BYTE PTR [rip+0x2f45],0x0        # 4030 <puts@plt+0x3000>
    10eb:	75 33                			jne    1120 <puts@plt+0xf0>
    10ed:	55                   			push   rbp
    10ee:	48 83 3d 02 2f 00 00 		          cmp    QWORD PTR [rip+0x2f02],0x0   # 3ff8 <puts@plt+0x2fc8>
    10f5:	00 
    10f6:	48 89 e5             			mov    rbp,rsp
    10f9:	74 0d                			je     1108 <puts@plt+0xd8>
    10fb:	48 8b 3d 26 2f 00 00 		          mov    rdi,QWORD PTR [rip+0x2f26]        # 4028 <puts@plt+0x2ff8>
    1102:	ff 15 f0 2e 00 00    		          call   QWORD PTR [rip+0x2ef0]               # 3ff8 <puts@plt+0x2fc8>
    1108:	e8 63 ff ff ff      		 	call   1070 <puts@plt+0x40>
    110d:	c6 05 1c 2f 00 00 01 		          mov    BYTE PTR [rip+0x2f1c],0x1           # 4030 <puts@plt+0x3000>
    1114:	5d                   			pop    rbp
    1115:	c3                   			ret    
    1116:	66 2e 0f 1f 84 00 00 		          cs nop WORD PTR [rax+rax*1+0x0]
    111d:	00 00 00 
    1120:	c3                   			ret    
    1121:	66 66 2e 0f 1f 84 00 		          data16 cs nop WORD PTR [rax+rax*1+0x0]
    1128:	00 00 00 00 
    112c:	0f 1f 40 00         		 	nop    DWORD PTR [rax+0x0]
    1130:	f3 0f 1e fa          			endbr64 
    1134:	e9 67 ff ff ff       			jmp    10a0 <puts@plt+0x70>
    1139:	55                   			push   rbp
    113a:	48 89 e5             			mov    rbp,rsp
    113d:	48 8d 05 c0 0e 00 00 		          lea    rax,[rip+0xec0]        # 2004 <puts@plt+0xfd4>
    1144:	48 89 c7            		 	mov    rdi,rax
    1147:	e8 e4 fe ff ff       			call   1030 <puts@plt>
    114c:	b8 00 00 00 00       		          mov    eax,0x0
    1151:	5d                   			pop    rbp
    1152:	c3                   			ret    

Disassembly of section .fini:

0000000000001154 <.fini>:
    1154:	f3 0f 1e fa          			endbr64 
    1158:	48 83 ec 08         	 		sub    rsp,0x8
    115c:	48 83 c4 08          		          add    rsp,0x8
    1160:	c3                   			ret 
	
```
Notou alguma diferença ? Perceba que nossa função `main`, se encontra na section `.text`, porém não a symbols identificando ela, chamadas de funções que estavam explicitos no disassembly não são mais visiveis, precisamos analisar o binário com mais atenção...

Para complementar o artigo, irei disponibilizar um video bastante interresante, no qual tem como intuito encontrar a `main` nos binários ELF, feito pelo
[mente binaria](https://www.youtube.com/c/PapoBin%C3%A1rio).

[![IMAGE ALT TEXT HERE](https://i3.ytimg.com/vi/63XrejF4sdg/hqdefault.jpg)](https://youtu.be/63XrejF4sdg)


<br>
> Citações do artigo
1. [Practical Binary Analysis 1st Edition](https://www.amazon.com.br/Practical-Binary-Analysis-Instrumentation-Disassembly/dp/1593279124)
2. [Programação moderna em C](https://www.youtube.com/watch?v=oZeezrNHxVo&list=PLIfZMtpPYFP5qaS2RFQxcNVkmJLGQwyKE)