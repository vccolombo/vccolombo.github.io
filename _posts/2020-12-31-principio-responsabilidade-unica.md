---
title: "Princípio da Responsabilidade Única"
excerpt: "Uma rápida introdução ao S do S.O.L.I.D."
date: 2020-12-31T11:25:30-03:00
categories:
  - Blog
tags:
  - OOP
  - SOLID
  - Portugues
---

O princípio da responsabilidade única diz que um módulo (seja classe, função, etc) deve ter uma única responsabilidade. Esse conceito foi criado por [Robert C. Martin](https://blog.cleancoder.com/uncle-bob/2014/05/08/SingleReponsibilityPrinciple.html), um dos grandes nomes da Engenharia de Software na atualidade.

O programador deve olhar para o código e verificar se a função está fazendo mais de uma coisa. Se estiver, deve ser separada em duas funções diferentes. Contudo, essa definição é muito aberta e confusa. Nem sempre é trivial definir se um trecho de código está fazendo mais de uma tarefa ou não.

A ideia é pensar no seguinte: se a lógica de uma tarefa X for modificada, o módulo que precisará ser alterado também causará impactos em outras tarefas Y e Z? Se sim, o Princípio da Responsabilidade Única está sendo ferido. ["O Princípio da Responsabilidade Única é sobre limitar o impacto de mudanças"](https://hackernoon.com/you-dont-understand-the-single-responsibility-principle-abfdd005b137). O foco então é minimizar os módulos que serão impactados por mudanças na [lógica de negócio](https://robsoncastilho.com.br/2016/03/10/logica-de-negocio-vs-logica-de-aplicacao/), correção de bugs, manutenção preventiva, ou qualquer alteração que se faça necessária no programa.

É interessante notar que, na verdade, o Princípio em si não tem a ver com o módulo fazer mais de uma coisa, e sim dele possuir mais de um motivo para ser modificado. Normalmente um é indicativo do outro, mas nem sempre. Isso é importante quando você olhar para uma função e estiver na dúvida se ela está "fazendo mais de uma coisa" ou não. As vezes elas estão atreladas de forma que há um único motivo para modificar a função, então o Princípio está sendo respeitado.

Contudo, é preciso tomar cuidado, pois tentar seguir o Princípio de forma descontrolada pode causar mais mal do que bem. Não queremos criar quantidades exageradas de módulos que tornem a navegação pelo código difícil. Quando criamos classes, queremos guardar dentro delas métodos que estão ligados ao propósito maior da classe, mas que podem não ter nada a ver entre si. Então a classe está fazendo mais de uma coisa? É preciso cautela na hora de decidir o que deve ser separado. Um exemplo simples pode ser que uma classe `Employee` deve ter que reportar o salário de um funcionário. Contudo, o cálculo do salário pode ser delagado a uma classe `Salary`, e deixar a `Employee` responsável apenas por retornar o salário de um empregado específico.

## Exemplo

No [site da minha Newsletter](https://www.freegamesnewsletter.tech/), há uma route para inscrever seu email. Veja o tamanho do código e o tanto de coisas que ele está fazendo:

```javascript
router.post("/subscribe", (req, res) => {
  var email = validator.unescape(req.body.email);
  if (!validator.isEmail(email)) {
    response = {
      responseCode: 1,
      responseMessage: "Invalid Email",
    };
    return res.status(400).json(response);
  }
  email = validator.normalizeEmail(email);

  const captchaResponse = req.body["g-recaptcha-response"][0];
  if (
    captchaResponse === undefined ||
    captchaResponse === "" ||
    captchaResponse === null
  ) {
    response = {
      responseCode: 1,
      responseMessage: "Please select captcha",
    };
    return res.status(400).json(response);
  }

  const verificationUrl =
    RECAPTCHA_URL +
    "&response=" +
    captchaResponse +
    "&remoteip=" +
    req.connection.remoteAddress;
  axios
    .get(verificationUrl)
    .then((response) => {
      if (response.data.success) {
        subscribersController.sendConfirmationEmail(email, (err, success) => {
          return res.status(200).json({
            responseCode: 0,
            responseMessage: "Success",
            redirect: "/checkYourEmail",
          });
        });
      } else {
        return res.status(400).json({
          responseCode: 1,
          responseMessage: "Failed Captcha",
        });
      }
    })
    .catch((error) => {
      console.log(error);
      return res
        .status(502)
        .json({ responseCode: 1, responseMessage: "Connection Error" });
    });
});
```

Essa função está responsável por:

- Verificar se o input do usuário está nos padrões para evitar entradas maliciosas.
- Normalizar o email.
- Verificar se o captcha é válido.
- Comunicar com o servidor do Google para checar o captcha.
- Dar seguimento com a subscrição.
- Responder a requisição do cliente com o resultado.

Quantas responsabilidades! A única tarefa que essa função deveria ter era servir como ponte entre a requisição do cliente e o trabalho no servidor. Nesse momento, se o Google modificar a forma como o Captcha funciona, pode causar necessidade de modificar também a parte que dá seguimento e a parte que responde o cliente.

Para começar a seguir o Princípio da Responsabilidade Única, as tarefas da lista devem ser divididas em módulos. Um código melhor então seria o seguinte:

```javascript
router.post("/subscribe", async (req, res) => {
  const { valid, errorMessage } = await validateRequest(req);
  if (!valid) {
    const response = {
      responseCode: 1,
      responseMessage: errorMessage,
    };

    if (errorMessage === "Connection Error") {
      return res.status(502).json(response);
    } else {
      return res.status(400).json(response);
    }
  }

  const { email } = normalizeParams(req.body);
  const successCallback = (err, success) => {
    return res.status(200).json({
      responseCode: 0,
      responseMessage: "Success",
      redirect: "/checkYourEmail",
    });
  };
  subscribersController.sendConfirmationEmail(email, successCallback);
});
```

Veja como o código é muito mais razoável. O corpo das tarefas foi passado para funções específicas (não mostradas nesse código por simplicidade): `validateRequest` para checar se os parâmetros recebidos do usuário são legítimos e se o captcha foi bem sucessido (essa função então também precisa ser dividida); `normalizeParams` para normalizar o email; `sendConfirmationEmail` para prosseguir com a requisição. A tarefa da rota passa a ser apenas receber a requisição e responder com o resultado do _backend_. Então, retomando o exemplo do Captcha, se o Google fizer alguma atualização no serviço, basta ir até a função que valida o Captcha e alterá-la. As mudanças no código estão confinadas em um único local, de forma que há maior segurança de que as modificações feitas não irão causar bugs em outras funções.

Talvez essa função possa ser modularizada ainda mais, com a criação de um método responsável por gerar as respostas, e a rota mostrada no código seria responsável apenas por chamar esse método e repassar a resposta para o cliente. Contudo, para esse projeto, considerei que isso seria [overengineering](https://www.codesimplicity.com/post/what-is-overengineering/), visto que é improvável que eu vá modificar a forma como as mensagens de resposta funcionam, e eu sou o único trabalhando no código atualmente.

## Conclusões

A conclusão que fica é que a maneira de se implementar códigos que seguem o Princípio da Responsabilidade Única não é uma receita de bolo perfeita e fácil de ser seguida. Depende do bom senso e da experiência do programador, além de pensamento cuidadoso e pautado nas decisões que fazem sentido para o projeto em si. Não é necessário deixar o código perfeito e prever todas as possibilidades futuras possíveis. Deve-se refletir o que se espera do projeto no futuro, e se houver necessidade de modificá-lo, o Princípio da Responsabilidade Única pode ser trabalhado e aperfeiçoado nessas mudanças, com o tempo.
