# Trabalho Testes
Diogo Oliveira Neiss

### Objetivo
Analisarei, nesse trabalho, três bibliotecas, em três diferentes linguagens. O foco será em testes de unidade, já que o entendimento é mais fácil e há menos carga cognitiva em entender o fluxo de testes

## 1. Caso: Fluent Validation: C# .NET

Para o primeiro comparativo de testes trabalharemos com o Fluent Validation. Ele serve para fazer validações de objetos mediantes regras estabelecidas e diferentes contextos programados, sendo uma excelente alternativa aos DataAnnotations do C#. Os testes em C# são utilizados com a biblioteca XUnit, comumente encapsulada em extensões/wrappers na biblioteca sendo testada.

Vamos analisar nessa seção uma classe de testes implementados pela biblioteca, disponível em [https://github.com/FluentValidation/FluentValidation/blob/main/src/FluentValidation.Tests/](https://github.com/FluentValidation/FluentValidation/blob/main/src/FluentValidation.Tests/)

A primeira parte da classe é a definição do validador, que criará as regras a serem validadas no código, a respeito de um cartão de crédito. Regras de validação foram incorporadas em outras classes, essa servirá para validar se sua verificação está boa.

A seguinte parte configura a classe e cria um validador, baseado na regra que o atributo creditCard atenda aos requisitos da extensão de validação.

    public  class  CreditCardValidatorTests {
    
	    TestValidator  validator;    
	    
	    public  CreditCardValidatorTests() {
	    
	    CultureScope.SetDefaultCulture();
	    
	      
	    
	    validator =  new  TestValidator {
	    
	    v  => v.RuleFor(x  => x.CreditCard).CreditCard()
	    
	    };
    
    }
Iremos agora analisar os testes unitários da classe

  

    [Fact] // copied these tests from the mvc3 unit tests.
    
    public  void  IsValidTests() {
    
    validator.Validate(new  Person { CreditCard =  null }).IsValid.ShouldBeTrue(); // Optional values are always valid
    
    validator.Validate(new  Person { CreditCard =  "0000000000000000" }).IsValid.ShouldBeTrue(); // Simplest valid value
    
    validator.Validate(new  Person { CreditCard =  "1234567890123452" }).IsValid.ShouldBeTrue(); // Good checksum
    
    validator.Validate(new  Person { CreditCard =  "1234-5678-9012-3452" }).IsValid.ShouldBeTrue(); // Good checksum, with dashes
    
    validator.Validate(new  Person { CreditCard =  "1234 5678 9012 3452" }).IsValid.ShouldBeTrue(); // Good checksum, with spaces
    
    validator.Validate(new  Person { CreditCard =  "0000000000000001" }).IsValid.ShouldBeFalse(); // Bad checksum
    
    }

Os testes criam um objeto person, com o cartão a ser testado, e em seguida criam um objeto validator recebendo-o como parâmetro. Em seguida, os testes devem refletir as regras de negócio.

Os métodos shouldBeTrue e ShouldBeFalse são wrappers para asserts clássicos do XUnit, biblioteca de testes do C#, aumentando sua expressividade. Podemos ver várioscasos de testes interessantes:

 - Valores nulos, "opcionais" em C#
 - Menor cartão válido possível
 - Cartão válido possível
 - Cartões válidos, porém com espaços e traços
 - Cartão quase válido, exceto por final

O teste poderia ser implementado de forma mais limpa, talvez utilizando a sintaxe de Theory do XUnit, que permite passar vários valores para uma única função, como se fosse um loop de chamadas com parâmetros variantes.

    [Fact]
    
    public  void  When_validation_fails_the_default_error_should_be_set() {
    
    string  creditcard  =  "foo";
    
    var  result  = validator.Validate(new  Person { CreditCard = creditcard });
    
    result.Errors.Single().ErrorMessage.ShouldEqual("'Credit Card' is not a valid credit card number.");
    
    }

O seguinte teste valida a mensagem de erro. É interessante observar o formato, ele segue bem o clássico Arrange-Act-Assert. Um cartão de crédito inválido é criado e uma pessoa com ele é criada e validada, forçando uma errorMessage dentro do objeto validator. Por fim, testa-se se o erro contido no validador é o erro esperado.

## 2. Caso: Alamofire:  Swift iOS

Alamofire é uma biblioteca de requisições http para Swift, com inúmeras possibilidades e features legais, visando dar mais segurança para aplicações. Analisaremos alguns testes da biblioteca, disponíveis em [https://github.com/Alamofire/Alamofire/blob/master/Tests/RequestTests.swift](https://github.com/Alamofire/Alamofire/blob/master/Tests/RequestTests.swift) .

O primeiro teste, bem simples, valida uma requisição get para um repositório mock.

    func  testRequestResponse() {
    
    // Given
    
    let url = Endpoint.get.url
    
    let expectation =  self.expectation(description: "GET request should succeed: \(url)")
    
    var response: DataResponse<Data?, AFError>?
    
      
    
    // When
    
    AF.request(url, parameters: ["foo":  "bar"])
    
    .response { resp in
    
    response = resp
    
    expectation.fulfill()
    
    }    
      
    
    waitForExpectations(timeout: timeout)   
      
    
    // Then
    
    XCTAssertNotNil(response?.request)
    
    XCTAssertNotNil(response?.response)
    
    XCTAssertNotNil(response?.data)
    
    XCTAssertNil(response?.error)
    
    }
Resumidamente, a aplicação faz uma chamada HTTP GET para o endpoint, passa dois parâmetros e espera ela se concluir. Por fim, é testado se requisição, resposta e dados são não nulos, e se também não houve nenhum erro.

Outro teste super interessante é o de requisições com parâmetros unicode, algo suportado pelo Swift e que deve ser suportado pela biblioteca também.

  

    func  testPOSTRequestWithUnicodeParameters() {
    
    // Given
    
    let parameters = ["french":  "français",
    
    "japanese":  "日本語",
    
    "arabic":  "العربية",
    
    "emoji":  "😃"]
    
      
    
    let expectation =  self.expectation(description: "request should succeed")
    
      
    
    var response: DataResponse<TestResponse, AFError>?
    
      
    
    // When
    
    AF.request(.method(.post), parameters: parameters)
    
    .responseDecodable(of: TestResponse.self) { closureResponse in
    
    response = closureResponse
    
    expectation.fulfill()
    
    }
    
      
    
    waitForExpectations(timeout: timeout)
    
      
    
    // Then
    
    XCTAssertNotNil(response?.request)
    
    XCTAssertNotNil(response?.response)
    
    XCTAssertNotNil(response?.data)
    
      
    
    if  let form = response?.result.success?.form {
    
    XCTAssertEqual(form["french"], parameters["french"])
    
    XCTAssertEqual(form["japanese"], parameters["japanese"])
    
    XCTAssertEqual(form["arabic"], parameters["arabic"])
    
    XCTAssertEqual(form["emoji"], parameters["emoji"])
    
    } else {
    
    XCTFail("form parameter in JSON should not be nil")
    
    }
    
    }

Semelhante ao exemplo acima, faremos uma requisição, dessa vez POST,  para um endpoint, passando o array de parametros chave/valor em unicode. Em seguida, é verificado se todos foram processados corretamente pela chamada HTTP, de modo que tenham dados dentro. Por fim, verificam-se os parâmetros da request, de modo que contenham os mesmos valores do array de parametros anterior, mantendo então a consistência unicode.

## 3. Caso: Numpy: Python

Analisaremos, por fim, uma biblioteca de operações numéricas/matriciais em Python super famosa, o Numpy.

Como os testes dependem de certos conhecimentos das operações do Numpy, exemplificarei testes simples e pouco acoplados.
Os arquivos podem ser encontrados em [https://github.com/numpy/numpy/tree/main/numpy/core](https://github.com/numpy/numpy/tree/main/numpy/core)

No arquivo test_arrayprint, são verificadas asserções bem importantes: formatação de arrays após manipulações do Numpy.


    def  test_summarize_1d(self):
    
    A = np.arange(1001)
    
    strA =  '[ 0 1 2 ... 998 999 1000]'
    
    assert_equal(str(A), strA)
    
      
    
    reprA =  'array([ 0, 1, 2, ..., 998, 999, 1000])'
    
    assert_equal(repr(A), reprA)
O teste acima é bem simples, é criado um array de 1001 elementos, começado em zero, e é comparado com sua forma pós conversão em string, que seria utilizando os três pontos no meio. A função testa a consistência dessa conversão.


    def  test_subclass(self):
    
    class  sub(np.ndarray): pass
    
      
    
    # one dimensional
    
    x1d = np.array([1, 2]).view(sub)
    
    assert_equal(repr(x1d), 'sub([1, 2])')
    
      
    
    # two dimensional
    
    x2d = np.array([[1, 2], [3, 4]]).view(sub)
    
    assert_equal(repr(x2d),
    
    'sub([[1, 2],\n'
    
    ' [3, 4]])')
    
      
    
    # two dimensional with flexible dtype
    
    xstruct = np.ones((2,2), dtype=[('a', '<i4')]).view(sub)
    
    assert_equal(repr(xstruct),
    
    "sub([[(1,), (1,)],\n"
    
    " [(1,), (1,)]], dtype=[('a', '<i4')])"
    
    )

Aqui outro teste, dessa vez comparando criação de diferentes tipos de array/matrizes e suas formas.
