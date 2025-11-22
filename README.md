# spring-cloud-config-sergio
Atividade de Arquitetura de Software - Spring Cloud Config


## a) Conceito de configuração externalizada e centralizada

Configuração **externalizada** é quando os parâmetros da aplicação (URL de serviços, credenciais, flags etc.) não ficam fixos dentro do código, e sim fora dele (arquivos YAML, variáveis de ambiente, Config Server). Isso permite mudar comportamento sem recompilar ou redeployar o código.

Configuração **centralizada** é quando vários serviços leem suas configurações de um ponto único, por exemplo um Config Server. Assim existe uma “fonte única da verdade” para as configs de todos os microserviços.

---

## b) Importância do Config Server em um banco digital com múltiplos ambientes

Em um banco digital existem vários ambientes (dev, homologação, produção), cada um com URLs, credenciais e integrações específicas. Manter essas configs duplicadas em cada serviço aumenta o risco de erro (por exemplo, apontar para o banco de produção em ambiente de teste).

Com um Config Server centralizado é possível:
- separar claramente as configs por ambiente (dev, hml, prod);
- alterar parâmetros de forma rápida em caso de incidente;
- padronizar as configs entre os serviços;
- reduzir a chance de expor credenciais sensíveis no código.

---

## c) Implementação realizada

- Foi criado um projeto **`config-server`** usando Spring Boot + Spring Cloud Config Server.
    - O servidor lê as configurações de um repositório Git:  
      `spring-cloud-config-sergio/config-repo`.
    - Arquivo de configuração para o serviço de contas (ambiente dev):  
      `config-repo/account-service-dev.yml`.

- Foi criado um cliente **`account-service`** que consome as configs do Config Server.
    - A aplicação usa:

      ```yml
      spring:
        application:
          name: account-service
        profiles:
          active: dev
        config:
          import: "configserver:http://localhost:8888"
      ```

    - Um controller simples (`MessageController`) expõe o endpoint:

      ```http
      GET /mensagem
      ```

      que retorna o valor da propriedade `message` definida no `account-service-dev.yml`.

- **Configurações por perfil (dev/prod)**  
  O arquivo `account-service-dev.yml` representa o ambiente de desenvolvimento.  
  Para produção, basta criar `account-service-prod.yml` com as configs de prod e trocar o perfil ativo para `prod`.

- **Refresh dinâmico sem restart**

  O controller está anotado com `@RefreshScope` e o Actuator está habilitado:

  ```yml
  management:
    endpoints:
      web:
        exposure:
          include: "*"
