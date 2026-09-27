# Suporte aos canais autorais tipo 24

As alterações foram adaptadas ao código desta pasta. Os arquivos anteriores à aplicação estão em `backup-before-type24`, com a mesma estrutura de diretórios.

## O que mudou

| Arquivo | Alteração |
| --- | --- |
| `Sunrise/src/middleware/bap/activity_message/scriptable_auth_body.h` | Estruturas dos canais tipo 24, limites do protocolo, controle de revisão e declarações do codec. |
| `Sunrise/src/middleware/bap/activity_message/activity_scriptable_auth_fixed_body_codec.cpp` | Codificação, decodificação e validação do corpo tipo 24; avanço da revisão sem estouro. |
| `Sunrise/src/server/activity/mission/mission_script_lua_slot_api.cpp` | Implementação e registro de `slot:set_authored_channels`. |
| `Sunrise/src/client/content/activity/activity_sdk_lua_contract.cpp` | Declaração da nova API no contrato Lua gerado. |
| `Sunrise/src/server/activity/activity_sdk_device_prepare.cpp` | Aceitação de mensagens tipo 24 válidas na preparação do comando. |
| `Sunrise/src/server/activity/host_runtime.h` | Novo tipo de comando, acrescentado ao final do enum para preservar os valores existentes. |
| `Sunrise/src/server/activity/host_runtime_internal.h` | Controle da última revisão tipo 24 por slot. |
| `Sunrise/src/server/activity/host_runtime_scriptable_requests.cpp` | Identificação e validação dos comandos tipo 24 na entrada do host. |
| `Sunrise/src/server/activity/host_runtime_scriptable.cpp` | Criação de uma revisão superior às revisões locais e retidas; aplicação da mesma revisão a todos os canais do comando. |
| `Sunrise/src/server/activity/host_runtime_scriptable_staging.cpp` | Confirmação da revisão após aceitação pelo transporte. |
| `Sunrise/resources/resource.h` | Metadados de versão alterados de `0.4.0.0` para `0.5.1.0`. |

Os comentários acrescentados estão em português do Brasil. Os comentários anteriores permanecem como estavam.

## Uso em Lua

```lua
-- O slot precisa ser um sensor autoral tipo 24, válido no SDK carregado.
-- O comando substitui o conjunto completo dos canais de saída.
local pedido = slot:set_authored_channels({
    values = {100, 0, -50},
    blend = 0,
})
```

`values` recebe de um a quatro valores finitos entre -100 e 100. `blend` é opcional, assume zero e deve ser finito e não negativo. A revisão é definida pelo host; o script não a escolhe. O comando segue a fila e a confirmação de transporte já utilizadas pelo projeto.

## Adaptações em relação ao patch original

- Esta árvore não tinha o codec básico tipo 24; ele foi incluído a partir do formato de três bits de contagem e registros de 96 bits, compatível com o catálogo existente.
- A API Lua foi colocada em `mission_script_lua_slot_api.cpp`, onde esta versão concentra as APIs de slots. Não foi necessário criar o arquivo separado usado pelo patch.
- A identidade do slot usa o tipo 24, o schema Auth `0x80804F40` e a associação exata de schemas produzida pelo SDK desta árvore, que resolve componente e Sense dinamicamente.
- O encaminhamento para o comando tipo 24 ocorre em `request_sdk_auth_override`; assim, não é necessário mudar a assinatura das APIs de despacho desta versão.
- A confirmação aceita uma revisão maior que o contador local quando o estado retido já estava à frente. Isso acompanha a lógica de composição do próprio patch e evita impedir a confirmação de um comando válido.

## Versão exibida

Antes da aplicação, `resource.h` já declarava `0.4.0.0`. Esse mesmo valor alimenta os metadados do Windows e a versão exibida na interface. O ajuste para `0.5.1.0` acompanha o nome desta pasta; não comprova, por si só, a origem ou a versão funcional de todos os arquivos do projeto.

## Testes

Execute `tests\run_type24_codec_test.cmd` para verificar o codec com o Visual Studio instalado. Os testes cobrem quantidades de canais, limites numéricos, valores não finitos, revisão negativa, estouro de revisão, buffer insuficiente, mensagem truncada, preenchimento inválido e um vetor de bytes independente.

A validação em jogo ainda precisa ser feita com um slot tipo 24 real do SDK correspondente.
