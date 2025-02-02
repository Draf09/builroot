# Implementacao de um servidor web escrito em Python
```
Autor: Rafael F. Dias 
Disciplina: Laboratório de Sistemas Operacionais
Professor: Miguel Xavier
Data: 10-04-2023 
Versao: 1.0
Licenca: GPL-2.0
Requisitos: python3
```

O objetivo da página HTML é fornecer informações básicas sobre o funcionamento do sistema (target - Buildroot) com as seguintes informações a ser apresentadas na página de maneira dinâmica:
- Data e hora do sistema;
- Uptime (tempo de funcionamento sem reinicialização do sistema) em segundos;
- Modelo do processador e velocidade;
- Capacidade ocupada do processador (%);
- Quantidade de memória RAM total e usada (MB);
- Versão do sistema;
- Lista de processos em execução (pid e nome).


Primeiramente será utilizado o simpleHttpServer para criar um servidor web em Python. O servidor web será executado na porta 8000 e o endereço IP será o localhost
192.168.1.10 (endereço IP do target) que servirá a página para o host (computador que está acessando o target).

O local onde se encontra o servidor é a pasta **/target/output/opt**. Nesta pasta, será criado um arquivo chamado server.py que será responsável por executar o servidor web.

Para executar o servidor web, basta executar o comando abaixo no terminal do target:
```python    
    python server.py
```

Data e hora do sistema

```python    
    get_datetime = subprocess.check_output(['sh', '-c', 'date "+%Y-%m-%d %H:%M:%S"'])
    datetime_str = get_datetime.decode().strip()
```

Uptime (tempo de funcionamento sem reinicialização do sistema) em segundos

```python  
    get_uptime = subprocess.check_output(['sh', '-c', 'awk \'{sub(/\..*/,"") ; print $1 }\' /proc/uptime'])
    uptime_str = get_uptime.decode().strip()
```

Modelo do processador e velocidade

```python  
    get_cpuinfo = subprocess.check_output(['sh', '-c', 'cat /proc/cpuinfo | grep "model name" | uniq | cut -d ":" -f 2'])
    cpuinfo_str = get_cpuinfo.decode().strip()
```


Capacidade ocupada do processador (%)
```python
with open('/proc/stat', 'r') as f:
    cpu_stats = f.readline().strip().split()[1:]
    cpu_total = sum(map(int, cpu_stats))
    cpu_idle = int(cpu_stats[3])
    cpu_usage = 100.0 - cpu_idle / cpu_total * 100.0
``` 


Quantidade de memória RAM total e usada (MB)

```python  
def get_meminfo():
    cmd = '''
        memtotal=$(awk 'NR==1 {printf \"%0.f\", $2/1000}' /proc/meminfo) && 
        memfree=$(awk 'NR==2 {printf \"%0.f\", ($1-$2)/1000}' /proc/meminfo) &&
        memused=$(( $memtotal - $memfree )) &&
        printf "%s,%s" "$memtotal" "$memused"
    '''
    p = subprocess.Popen(['sh', '-c', cmd], stdout=subprocess.PIPE, stderr=subprocess.PIPE)
    stdout, stderr = p.communicate()
    if p.returncode != 0:
        raise RuntimeError(f'Error running command: {cmd}.\n{stderr.decode()}')
    mem_str = stdout.decode().strip()
    memused, memtotal = mem_str.split(',')
    mem_str = f"Mem used: {memused}\nTotal mem: {memtotal} Mb"
    return mem_str
```



Lista de processos em execução (pid e nome)
```python  
get_cpu_model = subprocess.check_output(['sh', '-c', 'awk -F: \'/model name/ {print $2}\' /proc/cpuinfo | sed -n \'1p\''])
            cpu_model_str = get_cpu_model.decode().strip()

            get_cpu_speed = subprocess.check_output(['sh', '-c', 'awk -F: \'/cpu MHz/ {print $2}\' /proc/cpuinfo | sed -n \'1p\''])
            cpu_speed_str = get_cpu_speed.decode().strip()

            cpuinfo_str = f"CPU Model: {cpu_model_str}\nCPU Speed: {cpu_speed_str} MHz"    
```

Versão do sistema
```python
get_version = subprocess.check_output(['sh', '-c', 'cat /proc/version'])
version_str = get_version.decode().strip()
```

Lista de processos em execução (pid e nome)
```python   
get_proclist = subprocess.check_output(['sh', '-c', 'for i in /proc/[0-9]*/stat; do awk \'{gsub(/[()]/,""); printf "%s->%s,\\n", $1, $2}\' "$i"; done'])
proclist_str = get_proclist.decode().strip()
```

A página é exibida a partir do caminho **/target/output/opt/server.py** exibindo as informações do sistema de forma dinâmica.
    