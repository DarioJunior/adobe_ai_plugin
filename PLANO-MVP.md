# Plano do MVP — criação de vitrines com Illustrator, Photoshop e Claude

## Resumo

O MVP será um app executado localmente no Windows, conectado à Claude API pela internet e aos aplicativos Adobe por scripts. A pessoa seleciona uma arte do Illustrator e uma foto da fachada, descreve a direção visual desejada e recebe variações com dois resultados: arquivos `.ai` editáveis e prévias aplicadas à fachada.

A chave que o chefe forneceu serve para autenticar na Claude API se tiver sido criada no Console da Anthropic para a organização. Ela ficará somente no serviço local — nunca no código da interface, nos logs ou no repositório. As prévias reduzidas e as informações visuais necessárias serão enviadas à API; os arquivos originais permanecem locais. [Visão geral da Claude API](https://platform.claude.com/docs/en/api/overview)

## Etapas de implementação

### 1. Inspecionar os arquivos de exemplo

- Abrir o `.ai` no Illustrator e registrar pranchetas, camadas, grupos, textos, fontes, imagens vinculadas/incorporadas e objetos vetoriais.
- Verificar quais componentes já são editáveis individualmente. Elementos achatados ou rasterizados não podem ser recuperados automaticamente como vetores editáveis.
- Conferir dimensões, perspectiva e área da vitrine na fotografia.
- Resultado: teste de viabilidade com os materiais reais e lista de limitações da arte de entrada.

### 2. Construir o app local

- Criar uma interface para selecionar os arquivos, descrever a variação, escolher quantas propostas gerar e revisar os resultados.
- Manter um serviço local responsável por ler arquivos, chamar a API, validar a resposta e acionar scripts Adobe.
- Guardar a chave fora do código e da interface, com armazenamento local protegido quando viável. Não registrá-la em logs nem incluí-la no repositório.
- Usar um modelo Claude com visão disponível para a chave da organização; permitir configurar o identificador do modelo sem alterar o app. Acesso a modelos, limites e faturamento dependem da conta. [API, autenticação e limites](https://platform.claude.com/docs/en/api/overview)

### 3. Criar a ponte com o Illustrator

- Usar scripts locais `.jsx` para exportar uma prévia da arte e um inventário dos objetos relevantes.
- Identificar objetos e propriedades editáveis, como tipo, posição, tamanho, cor, texto e camada, sem enviar o documento `.ai` inteiro.
- Aplicar operações nos objetos existentes: mover, redimensionar, girar, recolorir, ocultar, duplicar, reorganizar e alterar texto.
- Permitir a criação de novos elementos vetoriais como grupos SVG importados para o Illustrator, separados e nomeados para edição independente.
- Construir cada variação em uma cópia própria para preservar o arquivo original.

### 4. Definir a resposta do Claude e validá-la

- Enviar à API uma prévia reduzida da arte, uma prévia da fachada, o inventário dos objetos e a instrução textual. Os arquivos originais não são enviados.
- Pedir dados estruturados contendo operações por ID de objeto, novos vetores em SVG e uma descrição curta da variação. Confirmar compatibilidade de saída estruturada com o modelo selecionado. [Saídas estruturadas](https://platform.claude.com/docs/en/build-with-claude/structured-outputs)
- Tratar a resposta como dados, nunca como código executável. Validar IDs, limites numéricos e SVGs; rejeitar scripts embutidos, referências externas e conteúdo ativo.
- Se a resposta for inválida ou não aplicável, não alterar a arte; mostrar o erro e permitir nova tentativa.

### 5. Montar as prévias no Photoshop

- Criar um script para aplicar a composição à área indicada da fachada, usando quatro pontos de perspectiva e uma máscara configurável.
- Manter foto e composição em camadas separadas no documento da prévia sempre que possível, facilitando ajustes manuais.
- Salvar uma imagem por variação e preservar a foto original.

### 6. Entregar o fluxo completo

- Gerar três variações iniciais por solicitação.
- Para cada variação, salvar um `.ai` editável e uma prévia da fachada, com nomes consistentes e os auxiliares necessários na pasta da execução.
- Permitir abrir os resultados no Illustrator e Photoshop para revisão humana; o app não publica nem substitui arquivos por conta própria.
- Começar com scripts executados pelo app local ou pelo comando de scripts do Adobe, não com um painel dentro do Illustrator. A Adobe documenta a execução de scripts no Illustrator e as operações sobre objetos. [Executar scripts no Illustrator](https://helpx.adobe.com/pt/illustrator/desktop/automate-visualize-data/automate-actions/install-and-run-scripts.html) · [Referência de objetos do Illustrator](https://ai-scripting.docsforadobe.dev/jsobjref/PathItem/)

## Testes e critérios de aceite

- A chave pode ser configurada localmente e não aparece no navegador, nos logs ou no repositório.
- Com uma chave autorizada, o app envia as prévias e recebe uma resposta validável. Falhas de rede, autorização ou limite são apresentadas sem perda dos arquivos locais.
- O arquivo `.ai` original permanece intacto após gerar as três variações.
- Cada resultado abre no Illustrator; objetos vetoriais novos e componentes editáveis da fonte podem ser selecionados e ajustados separadamente.
- Cada prévia corresponde à variação correta e fica alinhada à perspectiva da vitrine. A pessoa pode corrigir pontos e máscara quando necessário.
- Resposta malformada, ID inexistente ou SVG inseguro é rejeitado sem modificar a cópia de trabalho.
- O fluxo completo é validado no computador Windows com Illustrator e Photoshop instalados.

## Premissas e limites

- O primeiro provedor será a Claude API direta, usando uma chave organizacional fornecida e autorizada pelo responsável; confirmar os modelos e limites habilitados para essa chave.
- O app é local, mas as prévias reduzidas e o contexto visual necessário saem do computador para a Anthropic. Os arquivos originais permanecem locais.
- A IA pode alterar qualquer elemento e criar vetores; uma pessoa revisa os resultados antes de usá-los.
- A editabilidade depende da estrutura da arte original. Elementos rasterizados não se tornam automaticamente vetores; elementos novos serão importados como vetores.
- O projeto ainda precisa receber o arquivo `.ai` e a foto para a etapa de inspeção com os materiais reais.
