# Fase 3-Exploração e Teste de Identidade

### **Da Auditoria à Ação**

Após a Fase 2 confirmar que o domínio **PF-DC01** possui uma política de senhas fraca (mínimo de 7 caracteres), esta nova etapa foca em validar o impacto real dessa brecha. O objetivo mudou: saímos da observação passiva para a simulação de um ataque de **Password Spraying**, testando se a facilidade de criar senhas curtas compromete a segurança dos usuários.

<img width="785" height="165" alt="image" src="https://github.com/user-attachments/assets/40f4cd0f-ba6c-4b6e-819f-2b2e6dc52093" />

### **Preparação do Arsenal**

O sucesso de um ataque de *Password Spraying* depende da qualidade dos dados e da inteligência aplicada ao dicionário de senhas. A partir dos logs da fase anterior, realizei a curadoria manual para criar o ficheiro `users_final.txt`, removendo ruídos e prefixos para garantir que a ferramenta processasse apenas nomes de utilizadores válidos. Complementarmente, foi elaborada uma estratégia de **Dicionário Customizado**, utilizando termos relacionados com a identidade da empresa combinados com padrões de complexidade comuns, simulando o reconhecimento que um atacante real faria antes da intrusão.

<img width="886" height="253" alt="image" src="https://github.com/user-attachments/assets/1d9dcbec-ba82-468d-b5c6-15388b7506df" />

### **Sucesso na Exploração: Prova de Conceito**

Após a simulação do ataque de *Password Spraying*, validei que a política de senhas de 7 caracteres permitiu a utilização de credenciais previsíveis. O teste resultou no comprometimento da conta `carlos.dev`, confirmando que a vulnerabilidade de configuração (GPO) identificada na Fase 2 é explorável e representa um risco real ao domínio. o comando utilizado foi `nxc smb 192.168.1.150 -u carlos.dev -p 'Senha'`

<img width="1098" height="160" alt="image" src="https://github.com/user-attachments/assets/e85d6ee2-9823-439f-9308-8f9ea6283e04" />

### **Estabelecimento de Acesso Remoto (Shell)**

Com a credencial válida, utilizei o protocolo **WinRM (Windows Remote Management)** para estabelecer uma conexão interativa com o servidor através do comando`evil-winrm -i 192.168.1.150 -u carlos.dev -p 'Senha'` . Este passo simula a entrada de um atacante no ambiente interno, permitindo a execução de comandos diretamente no PowerShell do alvo.

<img width="1092" height="169" alt="image" src="https://github.com/user-attachments/assets/53f02b07-a4b8-478d-bccc-d9784884e1fc" />

### Enumeração Pós-Exploração

Já dentro do sistema como um usuário comum, executei comandos de reconhecimento para mapear o ambiente interno e verificar o nível de visibilidade que um invasor teria sobre a infraestrutura da **PixelForge**. `whoami` ; `hostname` ; `net user` ; `ipconfig` .

<img width="1086" height="312" alt="image" src="https://github.com/user-attachments/assets/1a04c295-39c0-4b25-9555-8a3ee041859c" />

### **Identificação de Vetores de Escalada de Privilégios**

Uma vez estabelecido o acesso com o utilizador `carlos.dev`, realizei uma auditoria aos privilégios da conta para identificar possíveis caminhos de elevação. A análise revelou uma configuração de segurança extremamente permissiva e crítica:

- **Privilégios Identificados:** A conta possui os privilégios **`SeImpersonatePrivilege`** e **`SeDebugPrivilege`** no estado **Enabled**.
- **Impacto Técnico:** O `SeImpersonatePrivilege` permite que um processo criado pelo utilizador assuma a identidade de qualquer cliente que se ligue ao sistema. Já o `SeDebugPrivilege` permite a inspeção e alteração de processos de nível de sistema

<img width="1096" height="469" alt="image" src="https://github.com/user-attachments/assets/77520961-5422-48fb-9fcb-8b7ffe4d9169" />

### **Escalada de Privilégios: Abuso de Privilégios de Debug**

Durante a fase de enumeração pós-exploração, identifiquei que a conta comprometida possuía o privilégio **`SeDebugPrivilege`** ativo. Em um ambiente Windows, este privilégio é destinado a engenheiros de sistemas e desenvolvedores para depurar processos de baixo nível, mas em mãos erradas, ele permite ignorar as permissões padrão do sistema operacional.

Diferente de ataques complexos que exploram falhas de código (Kernel Exploits), esta escalada baseou-se inteiramente em uma **falha crítica de configuração de segurança**. Utilizei o acesso administrativo indireto concedido pelo privilégio de depuração para redefinir a conta do **Administrador do Domínio**. O comando**:** `net user Administrator PixelForge@2026` foi executado com sucesso, garantindo o controle total (Full Domain Compromise) sobre o **PF-DC01**.

<img width="1006" height="100" alt="image" src="https://github.com/user-attachments/assets/d334b170-0592-4141-8434-1bd64d0692c3" />

### **Análise de Detecção e Telemetria (Blue Team)**

Finalizei a fase de exploração validando a visibilidade do ataque nos **Security Logs** do Windows Server. Através do **Event Viewer**, identifiquei os rastros digitais da intrusão: as múltiplas falhas de autenticação (**Evento 4625**) geradas pelo *Password Spraying*, o logon bem-sucedido (**Evento 4624**) e a evidência crítica de escalada de privilégios (**Evento 4724**), onde o utilizador `carlos.dev` registou a alteração de senha da conta `Administrator`. Esta análise confirmou que, apesar das falhas de configuração, o sistema gerou a telemetria necessária para uma resposta a incidentes eficaz.

<img width="1341" height="621" alt="image" src="https://github.com/user-attachments/assets/51d05140-dcf5-41ab-a949-c806d377c359" />

## **Conclusão e Próximos Passos**

Este projeto demonstrou que falhas críticas de configuração (GPOs permissivas e privilégios de depuração excessivos) podem levar ao comprometimento total de um domínio em poucos minutos. Como a infraestrutura de identidade foi dominada, as próximas etapas focarão na maturidade da segurança da **PixelForge**. Na **Fase 4**, implementaremos o *Hardening* do sistema e realizaremos um reteste para validar a eficácia das correções. Por fim, na **Fase 5**, conduziremos uma auditoria de conformidade com o **RGPD**, garantindo que o tratamento de dados sensíveis no servidor esteja alinhado com as melhores práticas de privacidade e exigências legais vigentes.
