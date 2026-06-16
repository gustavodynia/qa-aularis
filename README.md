# STLC aplicada ao app Aularis (TESTES EM PROGRESSO)

Este repositório contém a documentação do Ciclo de Vida de Testes de Software (STLC) aplicado ao pwa Aularis, um software em desenvolvimento, que tem como objetivo a organização de cronogramas e agendamentos de aulas para professores/mentores e alunos.

O objetivo do projeto é documentar meu aprendizado na área de QA, estruturar cenários e casos de testes em um software real.

---

## 1. Análise de Requisitos e Cenários de Teste (STLC - Fase 1 a 3)
Nesta etapa inicial do Ciclo de Vida de Testes (STLC), foram analisados os requisitos lógicos da funcionalidade de Autenticação (Login). Com base nessa análise, os cenários de teste foram especificados utilizando a metodologia **BDD (Behavior-Driven Development)** e a sintaxe **Gherkin**, servindo como roteiro para futuras automações.

### Funcionalidade: Autenticação de Usuários
#### Cenário 1: Login efetuado com sucesso
* **Objetivo:** Validar o acesso à plataforma com credenciais válidas de professor.

```gherkin
Cenário: Login efetuado com sucesso
  Dado que o usuário está na tela de login do Aularis
  Quando ele insere o e-mail "professor@aularis.dev"
  E insere a senha "Professor123!"
  E clica no botão "Entrar"
  Então o usuário deve ser redirecionado para o Painel Principal (Dashboard) do sistema
```

```gherkin
Cenário: Tentativa de login com formato de e-mail inválido
  Dado que o usuário está na tela de login do Aularis
  Quando ele insere o e-mail "professor@aularis"
  E insere a senha "Professor123!"
  E clica no botão "Entrar"
  Então o sistema deve exibir uma mensagem de erro informando que o formato do e-mail é inválido
  E não deve enviar a requisição de autenticação para o servidor back-end
```

```gherkin
Cenário: Tentativa de login com caracteres especiais inválidos no e-mail
  Dado que o usuário está na tela de login do Aularis
  Quando ele insere o e-mail "#$&*%@aularis.dev"
  E insere a senha "Professor123!"
  E tenta submeter o formulário
  Então o sistema deve barrar a submissão por falta de sanitização no front-end
  E deve alertar o usuário para utilizar caracteres válidos no campo
```

## 2. Planejamento e Configuração do Ambiente (STLC - Fase 4)

Para a execução dos testes, foi configurado um ambiente de homologação local (Localhost) idêntico ao de desenvolvimento, garantindo acesso completo aos logs do servidor e inspeção de rede.

* **Ambiente de Teste:** Localhost (`http://localhost:3000`)
* **Banco de Dados:** Instalação local gerenciada via Prisma ORM
* **Ferramentas de Suporte:** 
  * VS Code
  * Navegador DevTools (Aba Network para inspeção de payloads HTTP)
* **Stack de Automação:**
  * Python
  * Pytest
  * Playwright

## 3. Execução dos Casos de Teste (STLC - Fase 5)

Os cenários mapeados na Fase 1 foram executados manualmente no ambiente configurado. O comportamento de cada caso de teste e os resultados obtidos foram os seguintes:

* **CT-01 (Login com credenciais corretas):** **PASS** – O sistema validou os dados corretamente, gerou a sessão via NextAuth e redirecionou o usuário para a Dashboard inicial.
* **CT-02 (Formato de e-mail inválido):** **FAIL** – O sistema permitiu a envio do formulário sem validação de domínio superior (Gerou o `BUG-001`).
* **CT-03 (Caracteres especiais no e-mail):** **FAIL** – O input aceitou símbolos lógicos e disparou uma requisição desnecessária que estourou um stacktrace no servidor (Gerou o `BUG-002`).

## 4. Engenharia de Automação de Testes

Para otimizar o ciclo de regressão, iniciou-se a conversão dos cenários manuais em scripts de automação robustos utilizando a biblioteca **Playwright**. O foco inicial foi validar a integridade do fluxo básico de carregamento e interação da UI.

### Como executar a automação localmente

1. Garanta que as dependências do projeto estejam instaladas:
   ```pip install pytest-playwright```
1.1 Instale os binários dos navegadores usportados pelo Playwright:
   ```python -m playwright install```
2. Execute s suíte de testes em segundo plano:
  ```python -m pytest```
3. Para executar visualizando a interação da automação na tela:
  ```python -m pytest --headed    ```

## 5. Encerramento e Conclusão (STLC - Fase 6)

* **Total de Casos de Teste Planejados: 3** 
* **Total de Casos de Teste Executados: 3** 
* **Testes com Sucesso (PASS): 1** 
* **Bugs Encontrados (FAIL): 2** 

### Resumo da Execução:
A funcionalidade base de login cumpre seu papel principal de autenticação. No entanto, as falhas de validação no front-end fazem com que a infraestrutura receba solicitações de processamento desnecessárias no backend. Pontos que foram reportados, serão ajustados e cobertos pelas futuras automações de testes.

---

## 6. Relatório de Bug Encontrado (Bug Report)

### BUG-001: Ausência de Validação de Formato de E-mail (Multiplos domínios)
* **Gravidade:** Baixa (Melhoria de UX / Validação).
* **Componente:** Tela de login (`/`) - Campo de E-mail.

#### Descrição do Problema
O formulário de login aceita a submissão de e-mails com formatos incompletos ou estritamente inválidos, como a ausência do sufixo de domínio superior (ex: `professor@aularis` sem o `.com` ou `.dev`).

#### Passos para Reproduzir
1. Acesse o sistema em `http://localhost:3000`.
2. No campo de e-mail, insira: `professor@aularis`.
3. Insira qualquer senha e clique em **"Entrar"**.

#### Comportamento Atual vs. Esperado
* **Atual:** O sistema dispara uma requisição HTTP para o servidor, que tenta buscar essa string literal no banco de dados e retorna uma falha de autenticação após processar o dado.
* **Esperado:** O front-end deve impedir o clique usando uma validação nativa de formato de e-mail (`type="email"` ou validação Regex via Zod), exibindo um alerta imediato ao usuário antes de enviar dados à rede.

---

### BUG 002: Falta de Sanitização de Caracteres Especiais no Input
* **Severidade:** Baixa/Média (Otimização de Infraestrutura e Segurança)
* **Componente:** Tela de Login (`/`) - Campo de E-mail

#### Descrição
O campo de texto permite que caracteres especiais lógicos (`#$&*%`) sejam enviados diretamente na requisição de login. Isso força o provedor de credenciais do NextAuth a gerar erros internos controlados na tentativa de tratar esses símbolos na consulta ao banco de dados.

#### Passos para Reproduzir
1. Acesse o sistema em `http://localhost:3000`.
2. No campo de e-mail, insira exatamente: `#$&*%@aularis.dev`.
3. Insira a senha padrão e clique em **"Entrar"**.

#### Comportamento Atual vs. Esperado
* **Atual:** O front-end gera uma requisição `POST / 200 in 60ms`. O terminal do servidor evidencia um estouro de stacktrace com a falha `[auth][error] CredentialsSignin`.
* **Esperado:** O formulário deve sanitizar a entrada, limpando ou bloqueando caracteres especiais que não pertencem à estrutura padrão de um endereço de e-mail internacional (RFC 5322), evitando gasto desnecessário de recursos de processamento do servidor.