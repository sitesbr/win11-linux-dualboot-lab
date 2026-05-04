# Windows 11 + Linux Dual Boot com BitLocker, RAID 0 e TPM 2.0

Guia prático para preparar um computador com **Windows 11 Pro original**, **Linux em NVMe separado**, **TPM 2.0 ativo**, discos livres do **BitLocker** e **RAID 0 via software no Windows** para jogos, testes e benchmarks.

Este guia foi criado a partir de um cenário real de bancada, com foco em ajudar quem quer usar Windows e Linux no mesmo computador sem transformar o boot, os discos e a recuperação de dados em uma dor de cabeça.

---

## Objetivo

Este repositório mostra uma forma organizada de:

- manter o **TPM 2.0 ativo** para o Windows 11;
- instalar o **Linux em um NVMe separado**;
- evitar problemas com **BitLocker** em discos reaproveitados;
- descriptografar unidades antes de formatar, mover ou criar RAID;
- criar **RAID 0 via software no Windows** para jogos e testes;
- evitar RAID pela BIOS em cenários simples de dual boot;
- manter Windows e Linux separados para facilitar manutenção.

---

## Aviso importante

Este guia envolve operações que podem apagar dados.

Antes de fazer qualquer coisa:

- faça backup dos arquivos importantes;
- salve suas chaves de recuperação do BitLocker;
- confira com atenção as letras dos discos;
- confirme o tamanho dos discos antes de formatar;
- não apague partições se não tiver certeza absoluta;
- não use RAID 0 para arquivos importantes.

> **RAID 0 não é backup.**
>
> Se um dos discos do RAID 0 falhar, você pode perder todos os dados do volume.

Use RAID 0 apenas para:

- jogos;
- benchmarks;
- arquivos temporários;
- testes;
- bibliotecas que podem ser baixadas novamente.

---

## Cenário usado como referência

Hardware usado como base neste guia:

```text
Placa-mãe: ASUS TUF Gaming B550M-Plus
Sistema principal: Windows 11 Pro
Linux: NVMe separado
TPM: AMD fTPM / TPM 2.0 ativo
Modo de boot: UEFI
BitLocker: desligado nos discos de teste
RAID 0: feito via software no Windows
```

Organização dos discos no cenário real:

```text
C: Windows 11 Pro
E: Storage 2TB
F: SSD / Games
G: SSD
NVMe separado: Linux
```

Objetivo final:

```text
C: Windows 11 Pro
E: Storage / arquivos
F + G: RAID 0 via Windows para jogos e testes
NVMe separado: Linux
TPM 2.0: ativo
BitLocker: desligado nos discos de laboratório/teste
```

---

## O problema encontrado

Após reinstalar o Windows 11 Pro, alguns discos podem aparecer bloqueados pelo BitLocker.

Mesmo que o disco principal do Windows esteja normal, outros HDs ou SSDs podem ter vindo de uma instalação anterior e continuar criptografados.

Exemplo de situação:

```text
C: Windows sem BitLocker
F: Games sem BitLocker
E: Storage com BitLocker ativo
G: SSD com BitLocker ativo
```

Quando isso acontece, o Windows pode pedir a chave de recuperação do BitLocker ao acessar os discos.

Esse problema pode aparecer depois de:

- reinstalar o Windows;
- mudar a ordem de boot;
- alterar BIOS/UEFI;
- ativar ou desativar Secure Boot;
- preparar dual boot com Linux;
- mudar discos de lugar;
- trocar hardware;
- instalar outro sistema operacional;
- usar Rufus alterando requisitos da instalação;
- reaproveitar discos de outra instalação do Windows.

---

## Conceito principal

O problema não é o TPM 2.0.

O **TPM 2.0 pode continuar ativo** em uma instalação moderna do Windows 11. O que costuma causar dor de cabeça em dual boot, bancada de testes e troca de sistemas é o **BitLocker ativo** em discos que você pretende alterar, formatar, mover ou usar em outro ambiente.

Resumo:

```text
TPM 2.0: pode manter ligado
BitLocker: desligar nos discos de teste/laboratório
Linux: instalar em NVMe separado
RAID 0: usar apenas no Windows
RAID pela BIOS: evitar se o objetivo é simplicidade no dual boot
```

---

## Link para recuperar a chave BitLocker

Antes de mexer nos discos, acesse sua conta Microsoft e salve suas chaves de recuperação:

```text
https://aka.ms/myrecoverykey
```

A chave costuma ter 48 dígitos e pode aparecer em formato parecido com:

```text
123456-123456-123456-123456-123456-123456-123456-123456
```

Use o ID da chave exibido pelo Windows para encontrar a chave correta.

---

## Passo 1 — Abrir PowerShell como administrador

Clique com o botão direito no menu Iniciar e abra:

```text
Terminal do Windows (Administrador)
```

ou

```text
Windows PowerShell (Administrador)
```

---

## Passo 2 — Verificar o estado do BitLocker

Rode:

```powershell
manage-bde -status
```

Esse comando mostra quais discos estão:

- criptografados;
- descriptografados;
- bloqueados;
- desbloqueados;
- em processo de descriptografia;
- pausados.

Exemplo de disco já livre:

```text
Status da Conversão: Totalmente Descriptografado
Porcentagem Criptografada: 0.0%
Status de Proteção: Proteção Desativada
Status do Bloqueio: Desbloqueado
```

Exemplo de disco ainda em processo:

```text
Status da Conversão: Descriptografia em Andamento
Porcentagem Criptografada: 58.5%
```

Exemplo de disco pausado:

```text
Status da Conversão: Descriptografia em Pausa
```

---

## Passo 3 — Desbloquear discos protegidos

Se o disco estiver bloqueado, primeiro desbloqueie usando a chave de recuperação.

Exemplo para o disco E:

```powershell
manage-bde -unlock E: -RecoveryPassword SUA-CHAVE-DE-48-DIGITOS
```

Exemplo para o disco G:

```powershell
manage-bde -unlock G: -RecoveryPassword SUA-CHAVE-DE-48-DIGITOS
```

Depois confira:

```powershell
manage-bde -status
```

O disco precisa aparecer como:

```text
Status do Bloqueio: Desbloqueado
```

---

## Passo 4 — Desligar o BitLocker dos discos

Depois de desbloquear o disco, desligue o BitLocker.

Exemplo para o disco E:

```powershell
manage-bde -off E:
```

Exemplo para o disco G:

```powershell
manage-bde -off G:
```

O Windows deve responder algo parecido com:

```text
Descriptografia em andamento.
```

---

## Passo 5 — Acompanhar a descriptografia

Para acompanhar todos os discos:

```powershell
manage-bde -status
```

Para acompanhar apenas um disco:

```powershell
manage-bde -status G:
```

Espere até chegar em:

```text
Status da Conversão: Totalmente Descriptografado
Porcentagem Criptografada: 0.0%
```

Enquanto estiver em andamento, evite:

- desligar o PC;
- tirar da tomada;
- formatar disco;
- excluir partição;
- criar RAID;
- instalar Linux;
- rodar benchmark pesado;
- copiar grandes volumes de dados para o disco em processo.

---

## Passo 6 — Impedir o PC de suspender ou hibernar

Durante a descriptografia, é recomendável impedir o Windows de suspender ou hibernar.

Rode:

```powershell
powercfg /change standby-timeout-ac 0
powercfg /change hibernate-timeout-ac 0
```

Significado:

```text
standby-timeout-ac 0
```

Impede o PC de suspender automaticamente quando está ligado na energia.

```text
hibernate-timeout-ac 0
```

Impede o PC de hibernar automaticamente quando está ligado na energia.

O número `0` significa:

```text
Nunca
```

Depois que tudo terminar, você pode voltar para algo mais normal, por exemplo:

```powershell
powercfg /change standby-timeout-ac 60
powercfg /change hibernate-timeout-ac 0
```

Isso deixa o PC suspender após 60 minutos parado e mantém a hibernação desativada.

---

## Passo 7 — Pausar ou continuar a descriptografia

Se a descriptografia ficar pausada, aparecerá algo assim:

```text
Status da Conversão: Descriptografia em Pausa
```

Para continuar:

```powershell
manage-bde -resume E:
manage-bde -resume G:
```

Para pausar manualmente:

```powershell
manage-bde -pause E:
manage-bde -pause G:
```

Depois confira:

```powershell
manage-bde -status
```

O ideal é aparecer:

```text
Status da Conversão: Descriptografia em Andamento
```

---

## Passo 8 — Confirmar que terminou

Antes de formatar, criar RAID ou instalar Linux, confirme que os discos estão totalmente descriptografados.

Rode:

```powershell
manage-bde -status
```

O estado desejado é:

```text
Status da Conversão: Totalmente Descriptografado
Porcentagem Criptografada: 0.0%
Status de Proteção: Proteção Desativada
Status do Bloqueio: Desbloqueado
```

Depois reinicie o PC uma vez e rode novamente:

```powershell
manage-bde -status
```

Se continuar em 0.0%, o disco está pronto.

---

## Passo 9 — Conferir os discos antes de formatar

Antes de apagar qualquer volume, confira os discos pelo PowerShell:

```powershell
Get-Disk
Get-Volume
```

Preste atenção em:

- letra da unidade;
- tamanho;
- nome do volume;
- tipo do disco;
- se é SSD, HD ou NVMe;
- se é o disco certo.

Nunca apague um disco apenas pela letra se você não tiver certeza.

No cenário deste guia, os SSDs de RAID 0 eram:

```text
F: SSD / Games
G: SSD
```

Ambos com aproximadamente:

```text
223 GB cada
```

---

## Passo 10 — Criar RAID 0 via Windows

Neste guia, a recomendação é criar o RAID 0 pelo Windows, e não pela BIOS.

Motivos:

- evita complicar o dual boot;
- evita dependência de driver de RAID da placa-mãe;
- mantém o Linux separado;
- reduz risco de o instalador Linux mexer no arranjo;
- facilita manutenção e recuperação do Windows.

### Opção recomendada: Gerenciamento de Disco

1. Aperte:

```text
Win + X
```

2. Abra:

```text
Gerenciamento de Disco
```

3. Localize os dois SSDs corretos.

4. Confira o tamanho de cada um.

5. Não mexa no disco C do Windows.

6. Não mexa no NVMe do Linux.

7. Não mexa no disco de storage se ele tiver arquivos importantes.

8. Clique com o botão direito nos volumes dos SSDs que serão usados no RAID 0.

9. Escolha:

```text
Excluir Volume
```

10. Depois que os dois SSDs estiverem como espaço não alocado, clique com o botão direito no espaço não alocado.

11. Escolha:

```text
Novo Volume Listrado
```

12. Selecione os dois SSDs.

13. Formate em:

```text
NTFS
```

14. Dê um nome, por exemplo:

```text
RAID0_GAMES
```

15. Finalize.

O **Volume Listrado** é o equivalente ao RAID 0 via software no Gerenciamento de Disco do Windows.

---

## RAID 0: vantagens e riscos

Vantagens:

- soma a capacidade dos discos;
- pode melhorar leitura e escrita sequencial;
- útil para jogos, testes e benchmarks;
- bom para arquivos que podem ser reinstalados ou baixados novamente.

Riscos:

- não tem redundância;
- não protege dados;
- se um SSD falhar, o volume inteiro pode ser perdido;
- não deve ser usado para backup;
- não deve ser usado para arquivos únicos ou importantes.

Exemplo com dois SSDs de 223 GB:

```text
SSD 1: 223 GB
SSD 2: 223 GB
RAID 0 final: aproximadamente 447 GB
Segurança: nenhuma
Uso recomendado: jogos, testes e benchmarks
```

---

## Passo 11 — Configuração recomendada da BIOS

Para Windows 11 Pro + Linux em NVMe separado, a configuração recomendada é:

```text
Boot Mode: UEFI
CSM/Legacy: Desativado
TPM 2.0 / AMD fTPM: Ativado
SATA Mode: AHCI
Secure Boot: depende da distribuição Linux
RAID pela BIOS: não recomendado neste cenário
```

Na ASUS TUF Gaming B550M-Plus, o TPM costuma aparecer como:

```text
AMD fTPM
```

ou algo relacionado a:

```text
Firmware TPM
```

A ideia é manter o TPM 2.0 ativo para o Windows 11, mas evitar BitLocker nos discos que serão usados para testes, RAID, dual boot ou troca frequente de sistema.

---

## Passo 12 — Instalar Linux em NVMe separado

A forma mais segura para dual boot é instalar o Linux em um NVMe próprio.

Modelo recomendado:

```text
Disco do Windows: Windows 11 Pro
Disco RAID 0: apenas Windows / jogos / testes
NVMe separado: Linux
```

Passo a passo recomendado:

1. Termine toda a descriptografia do BitLocker.

2. Confirme com:

```powershell
manage-bde -status
```

3. Faça backup dos arquivos importantes.

4. Crie o RAID 0 no Windows, se for usar.

5. Desligue o PC.

6. Instale o NVMe novo para Linux.

7. Durante a instalação do Linux, escolha o NVMe separado.

8. Evite instalar Linux no disco do Windows.

9. Evite instalar Linux dentro do RAID 0 do Windows.

10. Use o menu de boot da placa-mãe para escolher entre Windows e Linux.

Em placas ASUS, o menu de boot normalmente é acessado com:

```text
F8
```

durante a inicialização.

---

## Por que não usar o RAID 0 do Windows no Linux?

Mesmo que o Linux consiga enxergar alguns arranjos de disco, não é recomendado depender de um RAID/software do Windows para usar no Linux.

Possíveis problemas:

- o Linux pode enxergar os discos separados;
- o volume pode depender de recurso específico do Windows;
- o instalador Linux pode confundir os discos;
- uma montagem incorreta pode colocar dados em risco;
- jogos em NTFS compartilhados entre Windows e Linux podem dar problemas com permissões e Proton;
- Fast Startup do Windows pode deixar volumes NTFS em estado inconsistente.

Para um dual boot mais limpo:

```text
Windows usa seus discos
Linux usa o NVMe dele
RAID 0 fica apenas no Windows
```

---

## Sobre jogos no Linux

Mesmo que você tenha jogos instalados no Windows, o ideal é manter uma biblioteca própria no Linux.

Recomendado para Linux:

```text
ext4
```

ou

```text
btrfs
```

Evite depender de uma biblioteca Steam/Proton compartilhada em NTFS com o Windows.

Isso reduz problemas com:

- permissões;
- prefixos do Proton;
- compatibilidade;
- arquivos travados;
- nomes de arquivos;
- montagem incorreta do Windows.

---

## Checklist antes de criar RAID 0

Use este checklist antes de apagar os SSDs:

```text
[ ] Tenho backup dos arquivos importantes
[ ] O disco C do Windows não será alterado
[ ] O disco E de storage não será alterado sem necessidade
[ ] O F está identificado corretamente
[ ] O G está identificado corretamente
[ ] O G chegou em 0.0% criptografado
[ ] O BitLocker está desligado nos discos
[ ] Sei que RAID 0 apaga os discos envolvidos
[ ] Sei que RAID 0 não é backup
[ ] Vou usar o RAID 0 apenas para jogos/testes/benchmarks
```

---

## Checklist antes de instalar Linux

```text
[ ] Windows 11 Pro instalado
[ ] TPM 2.0 ativo
[ ] Boot em UEFI
[ ] CSM/Legacy desativado
[ ] BitLocker desligado nos discos de teste
[ ] RAID 0 criado apenas para Windows
[ ] NVMe separado instalado para Linux
[ ] Backup feito
[ ] Pendrive de instalação Linux preparado
[ ] Disco correto escolhido no instalador Linux
```

---

## Checklist final esperado

```text
[ ] C: Windows 11 Pro funcionando
[ ] E: Storage funcionando
[ ] F + G: RAID 0 no Windows
[ ] NVMe separado: Linux
[ ] TPM 2.0 ativo
[ ] BitLocker desligado nos discos de laboratório
[ ] Dual boot funcionando pelo menu de boot
[ ] Windows e Linux separados por disco
```

---

## Comandos úteis

Ver status geral do BitLocker:

```powershell
manage-bde -status
```

Ver status de um disco específico:

```powershell
manage-bde -status G:
```

Desbloquear disco com chave de recuperação:

```powershell
manage-bde -unlock G: -RecoveryPassword SUA-CHAVE-DE-48-DIGITOS
```

Desligar BitLocker:

```powershell
manage-bde -off G:
```

Pausar descriptografia:

```powershell
manage-bde -pause G:
```

Continuar descriptografia:

```powershell
manage-bde -resume G:
```

Impedir suspensão automática:

```powershell
powercfg /change standby-timeout-ac 0
```

Impedir hibernação automática:

```powershell
powercfg /change hibernate-timeout-ac 0
```

Listar discos:

```powershell
Get-Disk
```

Listar volumes:

```powershell
Get-Volume
```

---

## O que evitar

Evite:

- instalar Windows 11 burlando TPM se sua placa-mãe já tem TPM 2.0;
- desativar TPM 2.0 sem necessidade;
- usar BitLocker em discos que você vive formatando ou movendo;
- criar RAID antes de descriptografar os discos;
- formatar disco sem backup;
- instalar Linux dentro do RAID 0 do Windows;
- usar RAID pela BIOS em um cenário simples de dual boot;
- compartilhar biblioteca de jogos do Windows com Linux em NTFS;
- usar RAID 0 para arquivos importantes.

---

## Conclusão

Para um computador de testes, jogos, Linux, Windows 11 Pro, benchmarks e produção de conteúdo, o caminho mais simples e seguro é:

```text
Windows 11 Pro em um disco
Linux em outro NVMe separado
TPM 2.0 ativo
BitLocker desligado nos discos de teste
RAID 0 via Windows apenas para jogos/testes
RAID pela BIOS evitado
Backup sempre em disco separado
```

Essa configuração reduz problemas com BitLocker, facilita dual boot, mantém o Windows 11 dentro dos requisitos modernos e deixa o Linux isolado em seu próprio NVMe.

O foco é simplicidade, recuperação fácil e menos dor de cabeça.
