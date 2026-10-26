# ToDoList com Spring Security

## Como executar
1. `./mvnw spring-boot:run`
2. Teste com Thunder Client/Postman usando *Authorization → Basic Auth* em `http://localhost:8080/tarefas`.

## Credenciais
| Usuário | Senha    | Perfil |
|---------|----------|--------|
| `user`  | `user123`  | USER  |
| `admin` | `admin123` | ADMIN |

## Permissões
| Operação | Sem auth | USER | ADMIN |
|---|---|---|---|
| GET /tarefas | 401 | 200 | 200 |
| GET /tarefas/{id} | 401 | 200 | 200 |
| POST /tarefas | 401 | 201/200 | 200 |
| DELETE /tarefas/{id} | 401 | 403 | 204 |

## Evidências
| # | Método | Endereço | Perfil | Resposta |
|---|---|---|---|---|
| 1 | GET | /tarefas | sem credenciais | 401 Unauthorized |
| 2 | GET | /tarefas | user com senha errada | 401 Unauthorized |
| 3 | GET | /tarefas | USER | 200 OK |
| 4 | POST | /tarefas (`{"titulo":"Estudar"}`) | USER | 200 OK |
| 5 | DELETE | /tarefas/1 | USER | 403 Forbidden |
| 6 | DELETE | /tarefas/1 | ADMIN | 204 No Content |

## Perguntas

**1. Autenticação x autorização?**
Autenticação identifica *quem* é o usuário (ex.: `user`/`user123` validado via HTTP Basic; falha = 401). Autorização define *o que* ele pode fazer (ex.: `user` pode listar e cadastrar tarefas, mas só `admin` pode excluir; falha = 403).

**2. Para que servem?**
- `SecurityFilterChain`: define a cadeia de filtros de segurança — mecanismo de autenticação (HTTP Basic), CSRF, sessão e regras de acesso por rota/método.
- `UserDetailsService`: fornece ao Spring Security os dados do usuário (nome, senha codificada, perfis) para autenticar. Aqui é um `InMemoryUserDetailsManager`.
- `PasswordEncoder`: codifica senhas e compara a senha enviada com o hash armazenado (BCrypt); senhas nunca ficam em texto puro.

**3. 401 x 403?**
401 Unauthorized: a requisição não está autenticada (sem credenciais ou credenciais inválidas). 403 Forbidden: o usuário foi identificado, mas não tem permissão (ou a requisição foi bloqueada, por exemplo por CSRF).

**4. Como as regras de acesso foram configuradas?**
Em `SecurityConfig`, via `authorizeHttpRequests`:
- `DELETE /tarefas/**` → `hasRole("ADMIN")`
- demais `/tarefas/**` → `hasAnyRole("USER", "ADMIN")`
- qualquer outra requisição → `authenticated()`

A ordem importa: a regra mais específica (DELETE) vem primeiro.

**5. O que é CSRF e como foi tratado?**
CSRF (Cross-Site Request Forgery) é um ataque em que um site malicioso faz o navegador de uma vítima já autenticada (via cookie de sessão) enviar requisições indesejadas. O Spring Security o bloqueia por padrão exigindo um token CSRF em POST/PUT/DELETE; sem ele, a resposta é **403 mesmo para um usuário com perfil correto** (por isso 403 nem sempre é falta de permissão). Neste projeto a API é stateless (`SessionCreationPolicy.STATELESS`), usa HTTP Basic no cabeçalho `Authorization` e não usa cookies de sessão, então o ataque não se aplica e o CSRF foi desabilitado (`csrf.disable()`). Sem isso, o POST do USER e o DELETE do ADMIN falhariam com 403 por falta de token.
