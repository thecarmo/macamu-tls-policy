# Política TLS de produção do MaCaMu

Este repositório público distribui **somente material público de confiança TLS**.
Não contém código do aplicativo, dados clínicos, credenciais ou chaves privadas.

## Política assinada de versão 2

`policy.json` contém o envelope Ed25519 assinado pelo responsável com a chave
`macamu-tls-2026a`. `public-keys.json` registra as duas chaves públicas de produção
e reserva; a confiança do app continua definida pelas chaves compiladas no
[PR #93](https://github.com/thecarmo/macamu/pull/93), sem baixá-las deste arquivo.
Nenhuma chave privada participa da publicação.

`policy.template.json` contém exatamente os bytes públicos assinados nesta
versão, para facilitar a revisão. **Não é um envelope assinado** e não deve ser
renomeado para `policy.json`.

A **versão 2** cobre o certificado de produção emitido em
18/09/2026 pela RNP ICPEdu GR46 OV TLS CA 2025, válido até 05/04/2027. A cadeia
foi validada para `transplante.virtual.ufc.br` usando somente as CAs já revisadas
no app, e os certificados RNP e GlobalSign R46 servidos correspondem aos do modelo.
A RNP passa a ser a referência ativa; as CAs anteriores são preservadas como
legadas. O SPKI da chave do servidor permanece o mesmo, em modo `observe`.

Essa versão tem atualização prevista para 03/10/2026 e expira em 17/11/2026.
Essas são as datas da **política**, independentes da validade do certificado.
Antes de assinar, revisar também as datas completas do JSON. Se a preparação
precisar ser renovada, usar a ferramenta `prepare_tls_policy.dart` com uma versão
superior à última publicada e à embarcada no app; nunca editar bytes já assinados.

O arquivo na raiz da branch `main` é distribuído anonimamente pela URL:

```
https://raw.githubusercontent.com/thecarmo/macamu-tls-policy/main/policy.json
```

O app recebe um envelope com `policy` (base64 dos bytes JSON assinados) e
`signatures` (assinaturas Ed25519). As chaves públicas aceitas são compiladas no
app: editar uma chave neste repositório não altera o que os aparelhos aceitam.
Não há token do GitHub no cliente.

Verificação desta publicação: assinatura Ed25519 conferida com OpenSSL e com o
verificador do app; o conteúdo assinado coincide byte a byte com o modelo.
SHA-256 do envelope `policy.json`:
`d461f076ac95a83a8ed94cbc1becd1ba860b04f07ed9502fb16f59e355becdf6`.
O consumo remoto nos aparelhos exige instalar uma versão do app que inclua
as chaves públicas e a ativação do bootstrap.

## Publicação

1. O responsável gera duas chaves privadas Ed25519 criptografadas, fora dos
   repositórios e do CI. Guarda backups separados e compartilha apenas as públicas.
2. No repositório do app, registra as duas públicas por PR e prepara uma política
   com versão maior que todas as anteriores. O runbook local é
   `docs/tls-trust-policy.md`; as ferramentas são `tool/generate_tls_policy_keys.sh`,
   `tool/prepare_tls_policy.dart` e `tool/sign_tls_policy.dart`.
3. Obtém a nova cadeia do servidor, revisa âncoras e datas e assina na máquina de
   custódia. A chave privada e sua senha nunca são colocadas neste repositório
   nem em secrets do GitHub Actions.
4. Abre um PR adicionando/atualizando `policy.json` com **a saída assinada**.
   O revisor verifica versão, destinos, CAs e janelas de atualização/validade.
5. Após integrar a política e as chaves públicas, executa a verificação da URL
   publicada e o ensaio com o aplicativo antes de alterar o certificado do servidor.

Não editar o conteúdo assinado depois de gerar o envelope. Não desativar a
validação TLS para contornar problemas. A troca de uma chave comprometida requer
nova versão do app que retire essa chave; a reserva protege contra perda, não
revoga automaticamente uma chave comprometida.

O GitHub é o canal de distribuição, e a assinatura é a autoridade do documento.
Cache/CDN, indisponibilidade e limitações de rede podem atrasar a atualização;
manter CAs de continuidade e publicar com antecedência continua necessário.
Uma segunda origem deve estar em provedor independente para trazer redundância.
