#Como colocar a mongo db em seu botJS
Requisitos 
<https://cloud.mongodb.com/> _Conta no mongo_ 
<https://www.npmjs.com/package/mongoose> _npm i mongoose_
<https://discordjs.guide/keyv/#command-handler> _Tutorial de bot Handler_

#Criando Conta

Ao entrar no site do MongoDB Atlas vá em "Sign-in" ou em "Try Free" e crie sua conta com os seus dados (Não precisa de cartão, é totalmente gratuito).
Logo após você criar a conta, o Cluster irá ser criado automáticamente, que é onde fica a sua Database.

Ok Agora, vamos começar a configurar!

1. Clique em Security, e depois vá a **DataBase Acess**
!(MongoDB)[https://cdn.discordapp.com/attachments/597994004088619018/599002678638673922/download.png]
Vá em **+ ADD NEW USER**

OBS: Lembre da senha q vc criou no **+ ADD NEW USER** Para funcionar a database

2- Agora precisamos liberar as conexões.
Na mesma aba do Security, tem a barrinha onde diz "Network Acess", vá nessa categoria e coloque o endereço de ip Global
!(MongoDB)[https://cdn.discordapp.com/attachments/597994004088619018/599002677979906081/download_1.png]

Pronto! já estabelecemos a conexão, criamos o usuário e agora é só conectar na database e usar ela.
Sabe a barrinha onde você clicou no Security?? Então, volte agora para a telinha inicial, no "Cluster" e clique em Connect:

!(MongoDB)[https://cdn.discordapp.com/attachments/597994004088619018/599002676805500928/download_2.png]

Irá aparecer 3 alternativas de uso, escolha a do meio. Onde faz a conexão com o aplicativo ("Connect Your Application")

!(MongoDB)[https://cdn.discordapp.com/attachments/597994004088619018/599002674397970452/download_3.png]

Após escolher uma das alternativas de como iremos usar a conexão através do mongo, agora só precisamos pegar o link gerado que o mongo nos dá (Não divulgue seu link para ninguém).

!(MongoDB)[https://cdn.discordapp.com/attachments/597994004088619018/599002673429086218/download_4.png]

Nisso você criou a Conta, fez o usuário, manteve as conexões abertas e agora pegou o link de conexão. O que falta mais? bom... usarmos isso no código né

#Aplicando o Mongoose no código**

Bom, lembra do nosso código base do bot usando a npm Discord.js? então, porque não usar ele como exemplo?
Crie um arquivo chamado "database.js", será onde irá ficar a conexão com a Mongo Atlas e o sistema de troca de documentos!

!(MongoDB)[https://cdn.discordapp.com/attachments/597994004088619018/599002671567077386/download_5.png]

Dentro do arquivo "database.js", iremos colocar o código onde fará a ligação com o Mongo Atlas, e a estrutura em schema do nosso documento (pequeno exemplo):

```js
var mongoose = require("mongoose") // faz requisicao do mongoose
var Schema = mongoose.Schema // só para deixar bonitinho
mongoose.connect("Link de sua conexão", { // Onde pegamos o link, da conexão em Cluster
    useNewUrlParser: true
}).then(function () { // Caso Logue Corretamente
    console.log('\x1b[32m[ BANCO DE DADOS ] \x1b[0mBanco de dados foi ligado');
}).catch(function () { // Caso de ERRO
    console.log('\x1b[31m[ BANCO DE DADOS ] \x1b[0mBanco de dados desligado por erro');
});

// Iremos fazer um pequeno databse do usuário:
var User = new Schema({
    _id: { type: String, required: true }, // ID Do Usuário
    rank: { type: Number, default: 1 }, // Rank do Usuário
    xp: { type: Number, default: 0 }, // Xp do Usuário
    money: { type: Number, default: 0 },
    coins:{type:Number,default: 0}
 // Dinheiro do Usuário
})
/* Colocamos assim, o nome do que queremos (que nem um Objeto sabe?)
money: { type: Number, default: 0 },
Type: será o formato dele (Number = Número, String = Texto, Boolean = true/false)
default: será o valor inicial que todos irão ter
required: è que é NECESSÀRIO TER
*/

// Aqui é a exportação. para usar essa base do documento, fora desse código (Troca de documentos)
var Users = mongoose.model("Users", User);
exports.Users = Users
```

Ok, temos o código onde faz a conexão com o Mongo e uma pequena base de um documento para o usuário, mas agora falta a gente fazer o uso disso!

Iremos criar um evento, em que quando o usuário enviar uma mensagem, o seu banco de dados será adicionado, ou irá ser modificado, com um pequeno sistema de Dinheiro, Xp e Level:

Adcione esse `code` dentro da index.js
```js
const Database = require("./database.js")

// Vamos fazer com que, a cada mensagem enviada pelo usuário, ele irá ganhar DINHEIRO XP e upará de nivel
grandle.on("message", message => {
    if (message.author.bot) return; // Retorna caso não seja bot
    // Database.Users ( Database é a conta que usamos ) e ( Users é a exportação que deixamos no código do "database.js")
    Database.Users.findOne({ "_id": message.author.id }, function (erro, documento) { // Procura na database o documento do usuário, pela id dele
        if (documento) { // Caso Encontre o documento, executará esse código:
            documento.coins += 1 // Dinheiro do usuário, será adicionado +1
            documento.xp += 10 // XP +10
            if (documento.xp > documento.level * 350) { // Caso o documento.xp teja chegado no limite dele, q é level*350
                documento.xp = 0 // o Xp vai pro 0
                documento.level += 1 // e o level sobe +1
            }
            documento.save(); // No fim de tudo, o documento será salvado
        } else { // Caso o Usuário não tenha ainda um documento salvo na Database
            new Database.Users({
                _id: message.author.id,
            }).save();
        }
    })
});
```

Bom, então é isso. É um pequeno tutorial de como criar, conectar e usar o Mongo Atlas.
