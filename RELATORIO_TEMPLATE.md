# 📝 Relatório do Laboratório 2 - Chamadas de Sistema

---

## 1️⃣ Exercício 1a - Observação printf() vs 1b - write()

### 💻 Comandos executados:
```bash
strace -e write ./ex1a_printf
strace -e write ./ex1b_write
```

### 🔍 Análise

**1. Quantas syscalls write() cada programa gerou?**
- ex1a_printf: 9 syscalls
- ex1b_write: 7 syscalls

**2. Por que há diferença entre os dois métodos? Consulte o docs/printf_vs_write.md**

```
    A diferença ocorre porque no caso do write(), ele sempre está chamando o syscall, já o printf não,
ele apenas faz a chamada de sistema quando por limite de linha, limite de tamanho, fim do programa e outros motivos. 

```

**3. Qual método é mais previsível? Por quê você acha isso?**

```
    O mais previsível é o write(), por um simples motivo que toda escrita ele chama o syscall
  independente de tudo, já o printf não, é imprevisível por conta dos motivos de chamada de sistema, que são variáveis.
```

---

## 2️⃣ Exercício 2 - Leitura de Arquivo

### 📊 Resultados da execução:
- File descriptor: 3
- Bytes lidos: 128

### 🔧 Comando strace:
```bash
strace -e openat,read,close ./ex2_leitura
```

### 🔍 Análise

**1. Qual file descriptor foi usado? Por que não começou em 0, 1 ou 2?**

```
foi utilizado o file descriptor 3, os files descriptors 0,1 e 2 não foram utilizados, pois são usados
para entrada padrão, saída padrão e erro padrão, não são utilizados para abrir um arquivo como é o exemplo
em questão.
```

**2. Como você sabe que o arquivo foi lido completamente?**

```
O arquivo foi lido corretamente, pela mensagem escrita na execução estar correta,e o número de bytes lidos, que segue o mesmo da mensagem e arquivo lido.
```

**3. Por que verificar retorno de cada syscall?**

```
Para verificar se houve algum erro na chamada de sistema, que implicaria a algum erro de escrita ou leitura.
```

---

## 3️⃣ Exercício 3 - Contador com Loop

### 📋 Resultados (BUFFER_SIZE = 64):
- Linhas: 25 (esperado: 25)
- Caracteres: 1062
- Chamadas read(): 21
- Tempo: 0.000100 segundos (média)

### 🧪 Experimentos com buffer:

| Buffer Size | Chamadas read() | Tempo (s) |
|-------------|-----------------|-----------|
| 16          |      82         |  0.000200 |
| 64          |      25         |  0.000100 |
| 256         |      6          |  0.000080 |
| 1024        |      2          |  0.000070 |

### 🔍 Análise

**1. Como o tamanho do buffer afeta o número de syscalls?**

```
Quanto maior o tamanho do buffer, menos chamadas de syscalls são necessárias.
```

**2. Todas as chamadas read() retornaram BUFFER_SIZE bytes? Discorra brevemente sobre**

```
Sim, primeiro que há a media de bytes lidos por read, o que seria impossivel calcular se a chamada
não retornasse o BUFFER_SIZE bytes, segundo, para o progama identificar possiveis erros, ele precisa retornar os bytes lidos, exemplo: caso o retorno fosse -1, ele iria retornar como erro.
```

**3. Qual é a relação entre syscalls e performance?**

```
Quanto menor o número de syscalls, melhor é a performance do progama, em relação ao tempo.
```

---

## 4️⃣ Exercício 4 - Cópia de Arquivo

### 📈 Resultados:
- Bytes copiados: 1364
- Operações: 6
- Tempo: 0.000206 segundos
- Throughput: 6466.17 KB/s

### ✅ Verificação:
```bash
diff dados/origem.txt dados/destino.txt
```
Resultado: [X] Idênticos [ ] Diferentes

### 🔍 Análise

**1. Por que devemos verificar que bytes_escritos == bytes_lidos?**

```
Porque é uma cópia, se um arquivo é cópia de outro, ambos devem ter o mesmo tamanho em memória.
```

**2. Que flags são essenciais no open() do destino?**

```
As flags essenciais são: o O_WRONLY(abre o arquivo apenas para escrita), O_CREAT(Cria um arquivo caso
não exista) e O_TRUNC(Trunca o arquivo já existente, limpa para não ocorrer duplicidade).
```

**3. O número de reads e writes é igual? Por quê?**

```
Não, apenas 1 read() foi efetuado, e 4 writes executados, isso se explica pela memória em cache que pode ter sido utilizada pela máquina durante a execução do programa, por isso só precisou ler uma vez o arquivo.
```

**4. Como você saberia se o disco ficou cheio?**

```
O terminal retornaria erro, pois haveria diferença de tamanho entre os arquivos, retornando "erro na escrita".
```

**5. O que acontece se esquecer de fechar os arquivos?**

```
Os dados que não foram gravados do buffer para o disco podem ser perdidos, ja que o sistema precisa de 
informações para gerenciar os recursos no computador.
```

---

## 🎯 Análise Geral

### 📖 Conceitos Fundamentais

**1. Como as syscalls demonstram a transição usuário → kernel?**

```
As syscalls demonstram essa transição de maneira simples, um processo requisita certa ação 
do processador e o kernel libera o processo para a CPU, então funciona como um pedido do processo 
para o kernel.
```

**2. Qual é o seu entendimento sobre a importância dos file descriptors?**

```
A importância do file descriptors se resume ao retornos dos syscalls, eles adiantam o trabalho do processador em verificar o estado atual do processo com os file descriptors.
```

**3. Discorra sobre a relação entre o tamanho do buffer e performance:**

```
Quanto maior o tamanho do buffer, maior será a performance, em relação ao tempo e desempenho, porem,
irá utilizar mais recursos de memória para isso.
```

### ⚡ Comparação de Performance

```bash
# Teste seu programa vs cp do sistema
time ./ex4_copia
time cp dados/origem.txt dados/destino_cp.txt
```

**Qual foi mais rápido?** cp

**Por que você acha que foi mais rápido?**

```
Acho que o cp foi mais rápido por ser um comando direto ao SO, em vez de um processo de usuário.
```

---

## 📤 Entrega
Certifique-se de ter:
- [X] Todos os códigos com TODOs completados
- [X] Traces salvos em `traces/`
- [X] Este relatório preenchido como `RELATORIO.md`

```bash
strace -e write -o traces/ex1a_trace.txt ./ex1a_printf
strace -e write -o traces/ex1b_trace.txt ./ex1b_write
strace -o traces/ex2_trace.txt ./ex2_leitura
strace -c -o traces/ex3_stats.txt ./ex3_contador
strace -o traces/ex4_trace.txt ./ex4_copia
```
# Bom trabalho!
