# Política TLS de produção do MaCaMu

Este repositório público distribui **somente material público de confiança TLS**.
Não contém código do aplicativo, dados clínicos, credenciais ou chaves privadas.

## Estado inicial: aguardando chaves públicas e primeira assinatura

`policy.template.json` é um modelo para revisão. **Não é uma política assinada**
e não deve ser renomeado para `policy.json`. A atualização remota permanece
inativa no app até receber as duas chaves públicas e publicar a primeira política
assinada. A versão inicial do app já contém as CAs GlobalSign R3 e RNP/GlobalSign R46.

URL planejada de distribuição anônima:

```
https://raw.githubusercontent.com/thecarmo/macamu-tls-policy/main/policy.json
```

O app recebe um envelope com `policy` (base64 dos bytes JSON assinados) e
`signatures` (assinaturas Ed25519). As chaves públicas aceitas são compiladas no
app: editar uma chave neste repositório não altera o que os aparelhos aceitam.
Não há token do GitHub no cliente.

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
