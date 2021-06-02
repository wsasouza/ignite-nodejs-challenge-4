# ignite-nodejs-challenge-4
Criação de uma API seguindo a estrutura de arquitetura limpa SOLID.

<h1 align="center">Introdução ao S.O.L.I.D</h1>

## Sumário
- [💻 Sobre o desafio](#-sobre-o-desafio)
- [🔗 Rotas da aplicação](#-rotas-da-aplicação)
  - [POST `/users`](#post-users)
  - [PATCH `/users/:user_id/admin`](#patch-usersuser_idadmin)
  - [GET `/users/:user_id`](#get-usersuser_id)
  - [GET `/users`](#get-users)
- [User model 👤](#-user-model)
- [Users repository 👥](#-user-repository)
- [Como foram realizado os testes? 🤔](#-como-foram-realizado-os-testes-)
- [Como usar ❓](#-como-usar)

## 💻 Sobre o desafio
Para fixar os nossos estudos sobre a arquitetura SOLID, foi proposto um desafio que deviamos fazer uma aplicação de listagem e cadastro de usuários seguindo os princípios.<br/> 
Uma das regras era que para que a listagem de usuários funcione, o usuário que solicita a listagem deve ser um admin (veremos mais detalhes de como foi feito).

## 🔗 Rotas da aplicação
### POST `/users`
A rota deveria receber, dentro do `body` da requisição, `name` e `email` e apenas cadastrar um usuário se o email não estiver em uso.
```
** Controller**

handle(request: Request, response: Response): Response {
    const { email, name } = request.body;

    try {
      const user = this.createUserUseCase.execute({ email, name });
      return response.status(201).json(user);
    } catch (err) {
      return response.status(400).json({ error: err.message });
    }
  }
```

```
** UseCase**

execute({ email, name }: IRequest): User {
    const userAlreadyexists = this.usersRepository.findByEmail(email);

    if (userAlreadyexists) {
      throw new Error("User already exists");
    }

    const user = this.usersRepository.create({ name, email });
    return user;
  }
```


### PATCH `/users/:user_id/admin`
A rota deveria receber, nos `params` da rota, o `id` de um usuário e transformar esse usuário em admin.
```
** Controller**

handle(request: Request, response: Response): Response {
    const { user_id } = request.params;

    try {
      const user = this.turnUserAdminUseCase.execute({ user_id });
      return response.json(user);
    } catch (err) {
      return response.status(404).json({ error: err.message });
    }
  }
```
```
** UseCase**

execute({ user_id }: IRequest): User {
    const user = this.usersRepository.findById(user_id);

    if (!user) {
      throw new Error("User not found");
    }

    const newUser = this.usersRepository.turnAdmin(user);
    return newUser;
  }
```

### GET `/users/:user_id`
A rota deveria receber, nos `params` da rota, o `id` de um usuário e devolver as informações do usuário encontrado pelo `body` da resposta.
```
** Controller**

handle(request: Request, response: Response): Response {
    const { user_id } = request.params;

    try {
      const user = this.showUserProfileUseCase.execute({ user_id });
      return response.json(user);
    } catch (err) {
      return response.status(404).json({ error: err.message });
    }
  }
```

```
** UseCase**

execute({ user_id }: IRequest): User {
    const user = this.usersRepository.findById(user_id);
    if (!user) {
      throw new Error("User does not found");
    }
    return user;
  }
```

### GET `/users`
A rota deveria receber, pelos `headers` da requisição, uma propriedade `user_id` contendo o `id` do usuário e retornar uma lista com todos os usuários cadastrados. O `id` deveria ser usado para validar se o usuário que está solicitando a listagem é um admin. O retorno da lista deve ser feito apenas se o usuário for admin.
```
** Controller**

handle(request: Request, response: Response): Response {
    const { user_id } = request.headers;
    try {
      const users = this.listAllUsersUseCase.execute({
        user_id: String(user_id),
      });
      return response.json(users);
    } catch (err) {
      return response.status(400).json({ error: err.message });
    }
```

```
** UseCase**

execute({ user_id }: IRequest): User[] {
    const user = this.usersRepository.findById(user_id);
    if (!user) {
      throw new Error("User not found");
    }
    if (!user.admin) {
      throw new Error("User does not have permission");
    }

    const users = this.usersRepository.list();
    return users;
  }
```
## User model 👤 
Para testar se aprendemos mesmo criar a estrutura dos `models`, também tinhamos que criar o `model` do user
```
**Meu Modelo**

class User {
  id: string;
  name: string;
  email: string;
  admin: boolean;
  created_at: Date;
  updated_at: Date;

  constructor() {
    if (!this.id) {
      this.id = uuidV4();
    }
    if (!this.admin) {
      this.admin = false;
    }
  }
}
```

## Users repository 👥
Para testar se aprendemos mesmo criar a estrutura dos `repositories`, também tinhamos que criar o `repository` do user.
```
** Meu código**

class UsersRepository implements IUsersRepository {
  private users: User[];

  private static INSTANCE: UsersRepository;

  private constructor() {
    this.users = [];
  }

  public static getInstance(): UsersRepository {
    if (!UsersRepository.INSTANCE) {
      UsersRepository.INSTANCE = new UsersRepository();
    }

    return UsersRepository.INSTANCE;
  }

  create({ name, email }: ICreateUserDTO): User {
    const user = new User();
    Object.assign(user, {
      name,
      email,
      created_at: new Date(),
      updated_at: new Date(),
    });
    this.users.push(user);
    return user;
  }

  findById(id: string): User | undefined {
    const user = this.users.find((user) => user.id === id);
    return user;
  }

  findByEmail(email: string): User | undefined {
    const user = this.users.find((user) => user.email === email);
    return user;
  }

  turnAdmin(receivedUser: User): User {
    const newUser = Object.assign(receivedUser, {
      admin: true,
      updated_at: new Date(),
    });
    return newUser;
  }

  list(): User[] {
    return this.users;
  }
}

```

## Como foram realizado os testes 🤔 ?
Em cada teste, tem uma breve descrição no que sua aplicação deve cumprir para que o teste passe.

Caso você tenha dúvidas quanto ao que são os testes, e como interpretá-los, dê uma olhada em **[nosso FAQ](https://www.notion.so/FAQ-Desafios-ddd8fcdf2339436a816a0d9e45767664)**

Para esse desafio, temos os seguintes testes:

### Teste do model

- **Should be able to create an user with all props**

    Para que esse teste passe, você deve completar o código do model de usuários que está em **src/modules/users/model/User.ts**.
    O usuário deve ter as seguintes propriedades:

```
{
  id: string,
  name: string,
  admin: boolean,
  email: string,
  created_at: Date,
  updated_at: Date,
}
```

Lembre que a propriedade `admin` deve sempre ser iniciada como `false` e o `id` deve ser um `uuid` gerado automaticamente.

### Testes do repositório

- **Should be able to create new users**

    Para que esse teste passe, é necessário que o método `create` do arquivo **src/modules/users/repositories/implementations/UsersRepository** permita receber o `name` e `email` de um usuário, crie um usuário a partir do model (que foi completado no teste anterior).

- **Should be able to list all users**

    Para que esse teste passe, é necessário que o método `list` do arquivo **src/modules/users/repositories/implementations/UsersRepository** retorne a lista de todos os usuários cadastrados na aplicação.

- **Should be able to find user by ID**

    Para que esse teste passe, é necessário que o método `findById` do arquivo **src/modules/users/repositories/implementations/UsersRepository** receba o `id` ****de um usuário e ****retorne o usuário que possui o mesmo `id`.

- **Should be able to find user by e-mail address**

    Para que esse teste passe, é necessário que o método `findByEmail` do arquivo **src/modules/users/repositories/implementations/UsersRepository** receba o `email` ****de um usuário e ****retorne o usuário que possui o mesmo `email`.

- **Should be able to turn an user as admin**

    Para que esse teste passe, é necessário que o método `turnAdmin` do arquivo **src/modules/users/repositories/implementations/UsersRepository** receba o objeto do usuário completo, mude a propriedade `admin` para `true`, atualize também a propriedade `updated_at`  e retorne o usuário atualizado.

### Testes de useCases

- **Should be able to create new users**

    Para que esse teste passe, é necessário que o método `execute` do arquivo **src/modules/users/useCases/createUser/CreateUserUseCase.ts** receba `name` e `email` do usuário a ser criado, crie o usuário através do método `create` do repositório e retorne o usuário criado.

- **Should not be able to create new users when email is already taken**

    Para que esse teste passe, é necessário que o método `execute` do arquivo **src/modules/users/useCases/createUser/CreateUserUseCase.ts** não permita que um usuário seja criado caso já exista um usuário com o mesmo email e, nesse caso, lance um erro no seguinte formato:

    ```tsx
    throw new Error("Mensagem do erro");
    ```

- **Should be able to turn an user as admin**

    Para que esse teste passe, é necessário que o método `execute` do arquivo **src/modules/users/useCases/turnUserAdmin/TurnUserAdminUseCase.ts** receba o `id` de um usuário, chame o método do repositório que transforma esse usuário em administrador e retorne o usuário após a alteração.

- **Should not be able to turn a non existing user as admin**

    Para que esse teste passe, é necessário que o método `execute` do arquivo **src/modules/users/useCases/turnUserAdmin/TurnUserAdminUseCase.ts** não permita que um usuário que não existe seja transformado em admin. Caso o usuário não exista, lance um erro no seguinte formato:

    ```tsx
    throw new Error("Mensagem do erro");
    ```

- **Should be able to get user profile by ID**

    Para que esse teste passe, é necessário que o método `execute` do arquivo **src/modules/users/useCases/showUserProfile/ShowUserProfileUseCase.ts** receba o `id` de um usuário, chame o método do repositório que busca um usuário pelo `id` e retorne o usuário encontrado.

- **Should not be able to show profile of a non existing user**

    Para que esse teste passe, é necessário que o método `execute` do arquivo **src/modules/users/useCases/showUserProfile/ShowUserProfileUseCase.ts** não permita que um usuário que não existe seja retornado. Caso o usuário não exista, lance um erro no seguinte formato:

    ```tsx
    throw new Error("Mensagem do erro");
    ```

- **Should be able to list all users**

    Para que esse teste passe, é necessário que o método `execute` do arquivo **src/modules/users/useCases/listAllUsers/ListAllUsersUseCase.ts** receba o `id` de um usuário, chame o método do repositório que retorna todos os usuários cadastrados e retorne essa informação.

- **Should not be able to a non admin user get list of all users**

    Para que esse teste passe, é necessário que o método `execute` do arquivo **src/modules/users/useCases/listAllUsers/ListAllUsersUseCase.ts** não permita que um usuário que não seja admin, acesse a listagem de usuários cadastrados na aplicação. Caso o usuário não seja admin, lance um erro no seguinte formato:

    ```tsx
    throw new Error("Mensagem do erro");
    ```

- **Should not be able to a non existing user get list of all users**

    Para que esse teste passe, é necessário que o método `execute` do arquivo **src/modules/users/useCases/listAllUsers/ListAllUsersUseCase.ts** não permita que um usuário que não exista, acesse a listagem de usuários cadastrados na aplicação. Caso o usuário não exista, lance um erro no seguinte formato:

    ```tsx
    throw new Error("Mensagem do erro");
    ```

### Testes das rotas

Para que esses testes passem, você deve fazer alterações em todos os controllers da aplicação. 

Você pode olhar qual controller recebe o conteúdo de qual rota observando o arquivo **src/routes/users.routes.ts**.

- **Rota - [POST] /users**
- **Rota - [PATCH] /users/:user_id/admin**
- **Rota - [GET] /users/:user_id**
- **Rota - [GET] /users**
    - **Should be able to list all users**

        Para que esse teste passe, usando o useCase apropriado, você deve permitir que a rota receba o `id` de um usuário **admin** pelo header `user_id` da requisição e retorne, no corpo da resposta, a lista dos usuários cadastrados.

    - **Should not be able to a non admin user get list of all users**

        **Should not be able to a non existing user get list of all users**

        Para que **esses dois testes** passem, caso algum erro tenha acontecido no useCase, retorne a resposta com status `400` e um json com um objeto `{ error: "mensagem do erro" }`, onde o valor da propriedade `error` deve ser a mensagem lançada pelo erro no useCase.


## Como utilizar ❓:
Primeiramente você deve baixar o projeto para sua máquina e acessar a pasta.
```
  ❯ git clone https://github.com/wsasouza/ignite-nodejs-challenge-4
  ❯ cd ignite-nodejs-challenge-4
```
E agora para baixar as dependências e rodar o projeto basta seguir o exemplo de acordo com o seu package manager <br/>

```
**yarn**  
  ❯ yarn
  ❯ yarn dev
```
```
**npm**
  ❯ npm install
  ❯ npm dev
```
