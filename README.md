# Nebula Engine

A Nebula Engine é um motor gráfico escrito em Java, ela funciona com qualquer linguagem de programação que tenha interoperabilidade com Java.

Este tutorial é focado em Java, irá tratar os conceitos básicos da ferramenta, ao final, teremos um jogo completo.


[Click aqui](https://github.com/senna-joao/Atividade/archive/refs/heads/main.zip) para fazer o download do projeto base.

### Scene da tela de título

O primeiro passo consiste na criação de uma scene, que é a classe onde todos os eventos do jogo acontecem. A primeira scene será a tela de título do jogo.

Essa classe irá implementar a interface Scene disponibilizada pela engine, o resultado deve ser como no código a seguir:


```java

public class TitleScreen implements Scene {

	@Override
	public void load() {}


	@Override
	public void create() {}


	@Override
	public void update() {}

}
```
Qualquer implementação da interface Scene deve ter os três métodos: load(), create(), update(). Dentro desses métodos tudo acontece.

No método load() é onde todos os assets são carregados para a memória, no caso da tela de título, será necessário apenas carregar um background.


```java

	@Override
	public void load() {
		loadImage("assets/background.jpg", "background");
	}

```

O método loadImage() permite carregar imagens estáticas e guardar na memória com um nome específico, no caso, "background". O próximo passo é mostrar essa imagem na tela.

Todos os _assets_ carregados são colocados na cena dentro do método create(), dentro desse método os objetos da _scene_ são criados. O primeiro objeto será apenas uma imagem como demonstrado a seguir:


```java

    @Override
	public void create() {
		new GraphicalObject("background", 0, 0);
	}

```

GraphicalObject é o tipo de objeto que não existe na simulação da física do jogo. Imagens meramente artísticas, sem colisão, são instâncias desse tipo de objeto. Os parâmetros do construtor desta classe são: nome salvo do assets na memória e as coordenadas X e Y na cena.

Na Nebula, os objetos são renderizados do topo superior esquerdo ao canto inferior direito, ou seja, para a imagem com o mesmo tamanho da cena preencher ela inteira, é necessário que ela seja carregada nas coordenadas x=0, y=0, por isso esses são os valores colocados no construtor do objeto.

O próximo passo para completar a parte visual da tela de título são os textos. Tanto o nome do jogo como o botão "play" serão feitos utilizando objetos do tipo Text.


```java

	new Text("BREAKOUT", 120, 100, 64);
	new Text("Play", 200, 500, 32);
```

A Nebula tem uma fonte padrão, então, basta instanciar com o texto sendo o primeiro parâmetro, as coordenadas X e Y e por fim o tamanho que ele será exibido.

É necessário o código da inicialização do jogo. Esse código irá rodar quando a tecla espaço for pressionada, para isso, a Nebula tem a função subscribeInputHandler(). Nessa função, todas as teclas pressionadas estão acessíveis.


```java

        subscribeInputHandler(pressedKeys -> {
			if (pressedKeys.contains(32)) {
                nextScene(new MainScene());
            }
		});

```

O número relativo à barra de espaço é 32, quando o jogador pressiona a barra de espaço, o código 32 estará presente na lista de teclas pressionadas, então irá chamar a função nextScene().

A função nextScene() recebe uma nova _Scene_ como parâmetro, que no caso deste jogo, será a classe principal onde o jogo irá acontecer.



### Scene do jogo

A classe que será a _Scene_ principal do jogo terá a mesma estrutura da classe utilizada para a tela de título, com a adição de três atributos.


```java

public class MainScene implements Scene {
    private Image ball;
    private int numBlocos = 0;
    private boolean running = false;
    

```

Esses atributos serão utilizados posteriormente no código.

Para os _assets_ desta _Scene_, a mesma imagem de _background_ será utilizada, além dessa imagem, mais três arquivos serão carregados para a memória.

```java

 @Override
  public void load() {
	  loadImage("assets/background.jpg", "background");
      loadImage("assets/ball.png", "ball" );
      loadImage("assets/plataforma.png", "plataforma" );
      loadSpriteSheet("assets/brick-sprite.png", "brick-sprite", 16, 16);
    }
```

Uma nova função foi utilizada, a função loadSpriteSheet(), essa função é a responsável por carregar os _frames_ de animação. Para a Nebula, o que diferencia uma imagem estática de uma animação é a forma que esse arquivo é carregado. 

A função loadSpriteSheet() armazena os dados em memória em blocos do tamanho passado por parâmetro, no caso do exemplo, 16x16 pixels.


No método create() será definido todo o jogo em si. Além do _background_ que será criado da mesma forma que na tela de título, também serão criados dois outros objetos, a bola e a plataforma.


```java

  this.ball = new Image("ball", 300, 800, 100, 0, BodyType.DYNAMIC);
  this.ball.getBody().setElasticityPercentage(100);

  var plataforma = new Image("plataforma", 300, 900, 100, 0, BodyType.KINEMATIC);
  var worldBounds = createWorldBounds(0, 0, 800, 1100);

```

O objeto que representará a bolinha do jogo é do tipo Image, objetos desse tipo participam da simulação de física. Para a bolinha, que precisa se mover e colidir com outros objetos, ela precisa ter um BodyType.DINAMIC, essa definição permite que o objeto se mova e consiga interagir com outros objetos na cena.

Para que a bolinha possa ser rebatida na cena, ela precisa ser elástica, então é necessário definir a sua elasticidade como 100% utilizando o método setElasticityPercentage().

No caso da plataforma que irá impedir que a bolinha caia, a definição deve ser BodyType.KINEMATIC, essa definição permite que a plataforma sofra colisão da bolinha mas não se mova em resposta à colisão, caso a definição fosse DYNAMIC, a bolinha iria transferir energia para a plataforma deslocando ela do lugar após cada colisão.

Também foram criados os limites da _scene_, na Nebula, por padrão, as _scenes_ são infinitas, mas como se trata de um jogo que depende de um espaço limitado, existe uma função capaz de criar limites para esse mundo, a função createWorldBounds(), essa função recebe como parâmetro os limites X e Y, criando uma "caixa" retangular do tamanho que o programador definir.

Uma das condições para final da partida é a bolinha "cair" da plataforma, para isso, é necessário adicionar um evento de colisão para o limite inferior do mapa.


```java

	worldBounds.get("bottom").addCollisionCallback(ball, () -> {
    	nextScene(new EndGameScene("GAME OVER!"));
    });
```


Para definir comportamentos de colisão, é necessário inscrever uma função de _callback_, neste caso, a função irá passar para a _scene_ do fim do jogo.

Como foi carregado um _spritesheet_ é necessário criar a animação que utilizará ele, um mesmo _spritesheet_ pode ser utilizado por diversas animações, mas no caso deste jogo, apenas uma animação será criada.


```java

createAnimation("brick-sprite", 0, 6, 15,  false, "sprite");
```

Para se criar a animação, além do nome do _spritesheet_, é necessário passar por parâmetro qual o primeiro _frame_, qual o último, a velocidade da animação, um booleano que representa se a animação irá se repetir, e um nome único para que essa animação seja acessível pelo resto do código.


Da mesma forma que na tela de título, é necessário definir os comandos dessa _scene_, as setinhas do teclado serão utilizadas para mover a plataforma, e a barra de espaço será utilizada pra começar a partida.


```java

subscribeInputHandler(pressedKeys -> {

	if (pressedKeys.contains(262) ) {
    	plataforma.setVelocityX(5);
    }
  	if (pressedKeys.contains(263)){
      plataforma.setVelocityX(-5);
    }

    if (pressedKeys.contains(32) && !this.running) {
		this.ball.setVelocityY(-5);
        this.running = true;
    }
    if (!pressedKeys.contains(262) && !pressedKeys.contains(263)) plataforma.setVelocityX(0);
});
```

O movimento da plataforma é realizado pela alteração da velocidade no eixo X, valores < 0 movem a plataforma para a esquerda, valores positivos movem a plataforma para a direita.
Quando nenhuma tecla está sendo pressionada, a velocidade X da plataforma será 0.

A simulação de física disponível na Nebula busca um comportamento realista, entretanto, para o caso deste jogo, esse realismo pode causar problemas, toda vez que a plataforma colidisse com a bolinha, ambas estando em movimento, os vetores dos movimentos poderiam somar velocidades e a bolinha poderia ser acelerada até velocidades impossíveis de controlar.

Para contornar esse problema, a _engine_ disponibiliza uma função que coloca uma velocidade constante em um objeto


```java

@Override
public void update() {
	setConstVelocity(this.ball, 5, 5);
}
```

A função foi utilizada dentro do método update() para que a velocidade seja ajustada em cada atualização. Além do objeto que terá sua velocidade ajustada, os dois outros parâmetros são os limites de velocidade X e Y.

Por fim, é necessário criar os blocos que irão ser quebrados pela bolinha ao decorrer do jogo, para isso, utilizaremos um método específico para encapsular esse comportamento.

```java
 private void buildMap() {
        var offset = 120;
        int[][] mapa ={% raw %}{{5,5,5,5,5,5,5,5,5,5,5,5,5,5,5},
                        {5,5,5,5,5,5,5,5,5,5,5,5,5,5,5},
                        {4,4,4,4,4,0,0,0,0,0,4,4,4,4,4},
                        {4,1,4,1,4,0,0,1,0,0,4,1,4,1,4},
                        {3,3,3,3,3,0,0,0,0,0,3,3,3,3,3},
                        {3,3,1,3,3,3,3,3,3,3,3,3,1,3,3},
                        {2,2,2,2,2,2,2,2,2,2,2,2,2,2,2},
                        {2,2,2,2,2,2,2,2,2,2,2,2,2,2,2}} {% endraw %};

        for (var i = 0; i < mapa.length; i++) {
            for (var j = 0; j < mapa[i].length; j++) {
                var num = mapa[i][j];

                if ( num != 0) {
                    var x  = j *38 +offset;
                    var y  = i *38;
                    var block = new Sprite("brick-sprite", x, y, BodyType.STATIC);

                    this.numBlocos++;

                    if (num == 2) {
                        block.setColor(50, 70, 20);
                    }
                    if (num == 3) {
                        block.setColor(80, 80, 40);
                    }
                    if (num == 5) {
                        block.setColor(255, 125, 0);
                    }
                    block.addCollisionCallback(this.ball, ()-> {
                       block.setCurrentanimation("sprite");
                       block.resume();
                       block.setEndAnimationCallback(block::destroy);

                       this.numBlocos--;

                       if (this.numBlocos < 1) {
                           nextScene(new EndGameScene("VITÓRIA!"));
                       }
                   });

                   block.setScale(2.5f);
               }
           }
        }
    }

```

_Loops_ e _arrays_ não fazem parte do escopo deste tutorial, então os pontos de atenção neste método são os relacionados diretamente com a _engine_. Os blocos criados são objetos do tipo Sprite, dessa forma, será possível renderizar a animação do tijolo quebrando. Também é possível modificar a coloração desses tijolos passando valores RGB por parâmetro do método setColor() de cada bloco.


Como feito anteriormente, também foi definido uma função de _callback_ para a colisão entre a bolinha e os tijolos, quando essa colisão ocorrer, será atribuída a animação ao objeto através do método setCurrentAnimation(). A animação fica pausada até que o método resume() seja chamado. Ao final da animação o bloco será destruído através do método setEndAnimationCallback(), método que permite que algum evento aconteça após o término de uma animação.

Por fim, é feita uma verificação da quantidade de blocos, quando o número de blocos chegar a 0, uma nova _scene_ será carregada, a _scene_ de fim da partida. 

Esse método de construção dos blocos deverá ser chamado dentro do método create() para que o cenário seja montado.


### Tela de fim de jogo

A última classe deste tutorial será a _scene_ de fim da partida, essa classe terá uma particularidade, ela terá um construtor que recebe uma _string_ como parâmetro, e essa _string_ será armazenada como atributo da classe.

```java

public class EndGameScene implements Scene {
    private String endText;

    public EndGameScene(String endText) {
        this.endText = endText;
    }
```

Dessa forma poderemos aproveitar a mesma classe tanto para o caso de vitória como de derrota. Da mesma forma que feito anteriormente, o _background_ carregado será o mesmo das cenas anteriores, e dentro do método create será criado o texto de fim de jogo.

```java

   @Override
    public void create() {
        new GraphicalObject("background", 0, 0);

        new Text(this.endText, 200, 100, 32);

        timeOut( 2000, () -> nextScene(new TitleScene()));
    }
```

Também foi utilizada a função timeOut(), essa função recebe dois parâmetros, a quantidade de milissegundos que deve aguardar e qual função deve ser executada, no caso, a função nextScene() passando a tela de título como parâmetro para que o jogo reinicie.

Neste momento o jogo deve estar completo.

A Nebula é uma engine em desenvolvimento, e caso termine o tutorial e o jogo não funcione corretamente, pode ser algum erro na ferramenta e não na sua programação.

