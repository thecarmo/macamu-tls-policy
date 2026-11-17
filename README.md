# Política TLS de produção do MaCaMu

Este repositório público distribui **somente material público de confiança TLS**.
Não contém código do aplicativo, dados clínicos, credenciais ou chaves privadas.

## Política assinada de versão 3

`policy.json` contém o envelope Ed25519 assinado pelo responsável com a chave
`macamu-tls-2026a`. `public-keys.json` registra as duas chaves públicas de produção
e reserva; a confiança do app continua definida pelas chaves compiladas no
[PR #93](https://github.com/thecarmo/macamu/pull/93), sem baixá-las deste arquivo.
Nenhuma chave privada participa da publicação.

`policy.template.json` contém exatamente os bytes públicos assinados nesta
versão, para facilitar a revisão. **Não é um envelope assinado** e não deve ser
renomeado para `policy.json`.

A **versão 3** renova a validade da política, preservando os destinos, as CAs,
as chaves de assinatura e o SPKI em observação da versão 2. Ela cobre o
certificado de produção emitido em
18/09/2026 pela RNP ICPEdu GR46 OV TLS CA 2025, válido até 05/04/2027. A cadeia
foi validada para `transplante.virtual.ufc.br` usando somente as CAs já revisadas
no app, e os certificados RNP e GlobalSign R46 servidos correspondem aos do modelo.
A RNP permanece como referência ativa; as CAs anteriores são preservadas como
legadas. O SPKI da chave do servidor permanece o mesmo, em modo `observe`.

| Campo | UTC | Fortaleza (UTC−3) |
| --- | --- | --- |
| Emissão (`issuedAt`) | 17/11/2026 20:34:24 | 17/11/2026 17:34:24 |
| Buscar atualização (`refreshAfter`) | 02/12/2026 20:34:24 | 02/12/2026 17:34:24 |
| Expiração (`expiresAt`) | 16/01/2027 20:34:24 | 16/01/2027 17:34:24 |

As datas completas, incluindo frações de segundo, estão no documento assinado.
São datas da **política**, independentes da validade do certificado. A versão 2
expirou em 17/11/2026 às 16:19:57 de Fortaleza. A versão 3 permite recuperar
clientes que já a haviam aceitado, sem apagar o controle de versão armazenado.
Os aparelhos precisam baixar e aceitar a nova política; a publicação não
garante adoção imediata nem substitui a validação em aparelhos físicos.

Revisar mensalmente e publicar a próxima política antes do vencimento, mesmo
quando o certificado não mudar. `refreshAfter` agenda buscas no app, mas não
renova a validade. A ferramenta `prepare_tls_policy.dart` usa atualização em
15 dias e validade de 60 dias; a próxima versão deve superar todas as anteriores
e a embarcada no app. Nunca editar bytes já assinados.

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
`860e28e55492411703ba93e7e233c94e52cf637685af1797583b64ee37cb1fec`.
O consumo remoto nos aparelhos exige instalar uma versão do app que inclua
as chaves públicas e a ativação do bootstrap.

## Publicação

Para renovar uma política, reutilizar uma das chaves já aceitas pelo app.
Gerar novas chaves não é parte da renovação de datas. Usar como entrada o payload
da última política verificada; o envelope externo não é a entrada da preparação.

1. O responsável gera duas chaves privadas Ed25519 criptografadas, fora dos
   repositórios e do CI. Guarda backups separados e compartilha apenas as públicas.
2. Na configuração inicial do app, registra as duas públicas por PR. Nas
   renovações, prepara uma política
   com versão maior que todas as anteriores. O runbook local é
   `docs/tls-trust-policy.md`; as ferramentas são `tool/generate_tls_policy_keys.sh`,
   `tool/prepare_tls_policy.dart` e `tool/sign_tls_policy.dart`.
3. Obtém a nova cadeia do servidor, revisa âncoras e datas e assina na máquina de
   custódia. A chave privada e sua senha nunca são colocadas neste repositório
   nem em secrets do GitHub Actions.
4. Abre um PR adicionando/atualizando `policy.json` com **a saída assinada**.
   O revisor verifica versão, destinos, CAs e janelas de atualização/validade.
5. Após integrar a política, executa a verificação da URL publicada e o ensaio
   com o aplicativo antes de alterar o certificado do servidor. Novas chaves
   públicas ou novos destinos exigem uma atualização revisada do app.

Não editar o conteúdo assinado depois de gerar o envelope. Não desativar a
validação TLS para contornar problemas. A troca de uma chave comprometida requer
nova versão do app que retire essa chave; a reserva protege contra perda, não
revoga automaticamente uma chave comprometida.

O GitHub é o canal de distribuição, e a assinatura é a autoridade do documento.
Cache/CDN, indisponibilidade e limitações de rede podem atrasar a atualização;
manter CAs de continuidade e publicar com antecedência continua necessário.
Uma segunda origem deve estar em provedor independente para trazer redundância.
