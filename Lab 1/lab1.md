

# Pentru rulare:

```sh-session
gcc hello.c -o hello
```

```sh-session
./hello
```

# Comenzi de bază în terminalul Linux:
	
	man command 	-	manualul de utilizare
	pwd		- 	directorul curent
	ls 		-	continutul directorului curent
	cp src tgt	- 	copiere fișiere
	mv src tgt	- 	mutare fișiere
	rm item 	- 	ștergere fișiere
	mkdir dir 	-	creare director
	rmdir dir 	-	ștergere director gol
	echo str 	-	repetare string
	cd path 	-	schimbă directorul curent


	. 		-	directorul curent
	.. 		-	directorul părinte
	cmd > file 	-	redirecționare ieșire către fisier
 	cmd1 | cmd2     -       pipe: legătură ieșire-intrare


	strace (strace ./hello)		- 	ne arată funcțiile de sistem (syscalls)
						necesare pentru a se executa executabilul
						(ktrace + kdump)
				
	ldd (ldd ./hello)		-	ne arată ce biblioteci sunt necesare pentru a executa
						fișierul executabil
						
	nm (nm ./hello)			-	simbolurile folosite de hello.c
	
### Pentru a vedea o comandă dintr-o secțiune anume

	$ man 1 write          - 	descrie write(1)
 	$ man 2 write          - 	descrie funcția de sistem (syscall-ul) write(2)

### Exemplu de sesiune 

 	$ pwd
	/ home / souser		- Afișează directorul curent de lucru (Print Working Directory), care este "/home/souser".
	
	$ touch foo		- Creează un fișier gol numit "foo" în directorul curent.
	
	$ ls
	foo			- Listează conținutul directorului curent. În acest moment, conține doar fișierul "foo".
	
	$ cp foo bar		- Copiază fișierul "foo" într-un nou fișier numit "bar" în același director.
	
	$ ls
	bar foo			- Listează conținutul directorului curent, care acum conține fișierele "foo" și "bar".
	
	$ mv bar baz		- Mută sau redenumește fișierul "bar" în "baz".
	
	$ ls
	baz foo			- Listează conținutul directorului curent, acum având fișierele "baz" și "foo".
	
	$ rm baz		- Șterge fișierul "baz" din directorul curent.
	
	$ ls
	foo			- Listează conținutul directorului curent, rămânând doar fișierul "foo".
	
	$ mkdir test		- Creează un nou director numit "test" în directorul curent.
	
	$ cd test /		- Încearcă să intre în directorul "test". Slash-ul final este greșit, dar poate fi ignorat de shell.
	
	$ pwd
	/ home / souser / test	- Afișează noul director curent, acum "/home/souser/test".
	
	$ cd ..			- Se deplasează înapoi în directorul părinte (un nivel mai sus).
	
	$ rmdir test		- Șterge directorul gol "test".
	
	$ echo hello
	hello			- Afișează textul "hello" în terminal.
	
	$ echo hello > hello . txt    - Creează un fișier numit "hello.txt" și scrie în el textul "hello".
	
	$ cat hello . txt
	hello                         - Afișează conținutul fișierului "hello.txt", care este "hello".


 
# Pentru GDB

 
```sh-session
gcc -g -O0 test.s -o test -no-pie
```


```sh-session
gdb ./test
```
	
	
	b symbol 	-	oprirea executiei la simbol
	p var 		-	tipareste valoarea variabilei
	n 		-	urmatoarea instructiune
	c 		-	continuarea executiei
	q 		-	iesire

	
			
	
