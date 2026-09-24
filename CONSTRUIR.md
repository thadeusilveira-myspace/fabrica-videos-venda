# Plano de obra — 7 marcos

Para o Claude Code de quem vai construir. **Um marco por vez, e cada um se prova antes do
seguinte.** O critério de aceite ou passa ou não passa — não existe "ficou bom".

Leia `PEDRAS.md` antes do Marco 0. Metade dos marcos tem uma armadilha esperando, e várias
delas não parecem bug.

---

## Escolhas de base (e por que elas)

**Renderização em React (Remotion) e não em ffmpeg puro.** O vídeo tem câmera que se move,
texto que entra com animação e botão que aparece no fim. Em ffmpeg isso vira um `filter_complex`
ilegível que ninguém consegue mudar depois. Em React é um componente com props. O preço a
pagar está inteiro na PEDRA 1.

**Transcrição local, não por serviço pago.** Uma live de 2 horas transcrita por API custa
dinheiro a cada tentativa, e você vai tentar dezenas de vezes. Whisper local roda de graça e
offline. O preço a pagar está na PEDRA 5.

**Sem banco de dados.** Uma lista em JSON por leva é a fonte da verdade e dá para abrir num
editor e entender. Um banco aqui seria peso sem retorno.

---

## Marco 0 — provar o ambiente antes de escrever o sistema

Não escreva a fábrica ainda. Prove que as três ferramentas funcionam nesta máquina.

1. Renderizar um vídeo de 3 segundos, tela preta, 1080×1920, com a palavra "teste" na tela.
2. Medir um arquivo de vídeo qualquer: duração, largura, altura, rotação, e o instante em que
   a faixa de vídeo e a de som começam.
3. Transcrever 60 segundos de áudio com o Whisper local e receber texto com tempo por trecho.

**Aceite:** os três rodam por linha de comando, sem interface gráfica, e devolvem resultado
que dá para ler num script.

> Gaste tempo aqui. Descobrir no Marco 4 que o seu ffmpeg não tem um filtro custa muito mais.

---

## Marco 1 — um trecho vira um vídeo

Corte um trecho (começo e fim em segundos) de uma gravação, e renderize esse trecho como
vídeo vertical com uma frase na tela.

**Aceite, medido e não olhado:**
- o arquivo existe;
- a contagem de quadros bate exatamente com a duração pedida;
- tem faixa de som **e o volume medido está entre −18 e −11 LUFS** (o alvo das redes é −14).

> O "e" da última linha é o ponto. Um conferidor que só pergunta "tem faixa de áudio?"
> aprova vídeo mudo — a faixa existe, só está em silêncio. Aconteceu.

---

## Marco 2 — a câmera se mexe sozinha

O vídeo é gravado em plano fixo. Quem assiste desiste em segundos. A câmera virtual é o que
transforma um corte de live em vídeo que prende.

Implemente, sobre o vídeo parado:
- **troca de plano a cada 3 a 6 segundos**, entre uns 4 tipos (aberto, médio, no produto,
  perto do rosto), cada um com o seu grau de aproximação e a sua altura de mira;
- **empurrão lento contínuo** durante o plano inteiro, com suavização na entrada e na saída;
- **"soco"**: aproximação curta e rápida em cima de palavra de venda ou número, e volta.

**Aceite:** um vídeo de 40 segundos tem entre 7 e 12 trocas de plano, o movimento não tem
solavanco, e **o zoom nunca corta a cabeça de quem fala** (veja PEDRA 6 — isso exige saber
onde o rosto está, não dá para fazer no chute).

---

## Marco 3 — a fala manda na câmera

A troca de plano não pode cair no meio de uma palavra. E o plano precisa combinar com o que
está sendo dito: quando a pessoa fala do produto, a câmera vai para o produto.

- transcreva o trecho **já cortado** (não a live inteira) com tempo por palavra;
- troque de plano **em fim de frase**;
- monte um vocabulário por tipo de produto: as palavras que indicam que ela está falando do
  produto, da qualidade, do tamanho, da garantia. Roupa e cosmético não usam as mesmas palavras,
  e sem isso a câmera só alterna aberto/médio sem sentido.

**Aceite:** nenhuma troca de plano cai dentro de uma palavra, e num trecho em que ela fala
do produto a câmera está no plano de produto.

---

## Marco 4 — a trava do assunto proibido

Esta é a razão de a fábrica existir. Sem isso, ela é um gerador de problema jurídico.

1. Transcreva a gravação inteira, com tempo.
2. Marque toda fala que toque no assunto proibido. Para preço: `R$`, "reais", número com
   centavos, "por apenas", "custa", "desconto", "cupom", "frete", "promoção", "só hoje",
   "últimas unidades". Marque também **nome próprio de cliente** (a pessoa chama gente do
   chat pelo nome) e **promessa de prazo**.
3. Escolha janelas de 28 a 50 segundos que **não encostem** em nenhuma fala marcada.
4. Gere os vídeos.
5. **Transcreva cada arquivo FINAL de novo e varra.**

**Aceite:** o passo 5 dá zero. E, de propósito, apague um dos arquivos e rode de novo: a
mensagem tem que ser "falta arquivo", nunca "fala preço" (LEI 4).

> Atenção à PEDRA 13 antes de escrever a busca de palavras. Ela parece bobagem e reprova
> vídeo limpo.

---

## Marco 5 — o lote e o relatório que mede

Um vídeo leva 2 a 3 minutos para renderizar. Uma leva de 13 leva quase uma hora.

- rode o lote **destacado do terminal**, de modo que fechar a sessão não mate o trabalho
  (PEDRA 2);
- **congele uma cópia do código** antes de começar e renderize a partir dela (PEDRA 3);
- no fim, gere um relatório que **mede** cada arquivo: quadros, som em LUFS, quantas trocas
  de plano, onde entram as emendas.

**Aceite:** feche o terminal no meio do lote e o lote continua. E o relatório reprova sozinho
um arquivo que você estragou de propósito.

---

## Marco 6 — a entrada automática

Uma pasta é vigiada. Quem manda o material joga a gravação lá e não avisa ninguém.

- só começa quando o arquivo **parou de crescer** por um tempo — e cuidado com a PEDRA 4,
  que faz o detector nunca disparar;
- descubra o produto **ouvindo** a gravação, não pelo nome do arquivo: numa live de vendas a
  palavra mais repetida que não é palavra de ligação é o produto (cuidado: "cheiro" aparece
  mais que "perfume" e não é o produto — qualidade não é produto);
- escreva o estado **dentro da pasta de quem mandou** (LEI 3);
- o que sair vai para a pasta de espera, e só sai de lá por comando (LEI 2).

**Aceite:** largue uma gravação na pasta e vá embora. Sem tocar em nada, os vídeos aparecem
na pasta de espera com relatório e prova. E uma pasta com um arquivo que o sistema não lê
recebe explicação escrita em menos de um minuto.

---

## Marco 7 — o formato que prende

Só depois que tudo acima funciona. Está em `PADRAO-DE-EDICAO.md`: o vídeo que abre por um
**momento** da live em vez de abrir por uma mensagem. É o que mais rendeu, e é a parte que
mais depende de gosto — por isso fica por último, quando a fábrica já é confiável.

---

## O que NÃO construir

- **Banco de dados.** JSON por leva basta e é legível.
- **Interface para editar vídeo.** Quem ajusta, ajusta a lista JSON. Construir um editor é
  outro produto, dez vezes maior.
- **Postagem automática.** Fere a LEI 2, e o risco de uma máquina postando sozinha numa conta
  de loja não compensa nada.
- **Escolha de música.** Vídeo com link de produto no TikTok Shop só pode usar som da
  biblioteca comercial da própria plataforma. Música embutida no arquivo faz o vídeo ser
  silenciado. Os vídeos saem sem música, de propósito; a trilha se escolhe na hora de postar.
