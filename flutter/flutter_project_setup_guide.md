# Guia de Inicialização de Projeto Flutter

## 1. Estrutura Base do Projeto

```
lib/
├── src/
│   ├── controllers/    # Controladores e lógica de negócios
│   │   ├── bloc/      # Implementações BLoC
│   │   ├── cubit/     # Implementações Cubit
│   │   └── services/  # Serviços da aplicação
│   │
│   ├── data/          # Camada de dados
│   │   ├── datasources/  # Fontes de dados (local/remoto)
│   │   ├── models/    # Modelos de dados
│   │   └── repositories/ # Implementações de repositórios
│   │
│   ├── domain/        # Regras de negócio
│   │   ├── entities/  # Entidades do domínio
│   │   ├── repositories/ # Interfaces de repositórios
│   │   └── usecases/  # Casos de uso
│   │
│   ├── infrastructure/# Configurações e serviços
│   │   ├── config/    # Configurações da aplicação
│   │   ├── services/  # Serviços de infraestrutura
│   │   ├── utils/     # Utilitários
│   │   └── routes/    # Configuração de rotas
│   │
│   └── ui/            # Interface do usuário
│       ├── screens/     # Páginas da aplicação
│       │   ├── pages/    # Nome da pagina
│       │   └── viewmodel/    # Arquivo com a lógica e regras de negócios
│       ├── widgets/   # Widgets reutilizáveis
│       └── themes/    # Temas e estilos
│
├── main.dart          # Ponto de entrada
└── firebase_options.dart # Configurações do Firebase
```

## 2. Dependências Essenciais

Adicione ao `pubspec.yaml`:

```yaml
dependencies:
  flutter:
    sdk: flutter

  # Gerenciamento de Estado
  bloc: ^9.0.0
  flutter_bloc: ^9.0.0

  # Injeção de Dependência
  get_it: ^8.0.3

  # Navegação
  go_router: ^14.1.4

  # UI e Design
  responsive_framework: ^1.4.0
  google_fonts: ^6.2.1
  adaptive_theme: ^3.6.0

  # Armazenamento
  hive: ^2.2.3
  hive_flutter: ^1.1.0

  # Firebase
  firebase_core: ^3.3.0
  firebase_analytics: ^11.2.1
  firebase_crashlytics: ^4.0.4

  # Utilitários
  equatable: ^2.0.5
  intl: ^0.19.0
  flutter_localizations:
    sdk: flutter

dev_dependencies:
  flutter_test:
    sdk: flutter
  bloc_test: ^10.0.0
  build_runner: ^2.4.8
  hive_generator: ^2.0.1
```

## 3. Configuração Inicial

No arquivo `main.dart`:

```dart
import 'package:flutter/material.dart';
import 'package:get_it/get_it.dart';

import 'src/infrastructure/config/app_config.dart';
import 'src/infrastructure/services/app_service.dart';
import 'src/controllers/controllers.dart';
import 'src/data/data.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();

  // Inicialização de serviços
  await initServices();

  // Inicialização de controladores
  initControllers();

  // Inicialização da aplicação
  runApp(const MyApp());
}
```

## 4. Padrões de Nomenclatura

1. **Arquivos**:
   - Use snake_case para nomes de arquivos
   - Exemplo: user_repository.dart, login_page.dart

2. **Classes**:
   - Use PascalCase para nomes de classes
   - Exemplo: UserRepository, LoginPage

3. **Variáveis e Métodos**:
   - Use camelCase para variáveis e métodos
   - Exemplo: getUserData(), currentUser

4. **Constantes**:
   - Use SCREAMING_SNAKE_CASE para constantes
   - Exemplo: MAX_RETRY_COUNT, API_BASE_URL

## 5. Estrutura de BLoC

```dart
// Exemplo de estrutura BLoC
class UserBloc extends Bloc<UserEvent, UserState> {
  final UserRepository _repository;

  UserBloc(this._repository) : super(UserInitial()) {
    on<LoadUserEvent>(_onLoadUser);
  }

  Future<void> _onLoadUser(
    LoadUserEvent event,
    Emitter<UserState> emit,
  ) async {
    try {
      emit(UserLoading());
      final user = await _repository.getUser(event.userId);
      emit(UserLoaded(user));
    } catch (e) {
      emit(UserError(e.toString()));
    }
  }
}
```

## 6. Estrutura de Repositório

```dart
// Interface
abstract class IUserRepository {
  Future<User> getUser(String userId);
  Future<void> saveUser(User user);
}

// Implementação
class UserRepository implements IUserRepository {
  final IUserDataSource _dataSource;

  UserRepository(this._dataSource);

  @override
  Future<User> getUser(String userId) async {
    return await _dataSource.getUser(userId);
  }
}
```

## 7. Estrutura de UI

```dart
// Exemplo de estrutura de página
// lib/src/ui/screens/pages/user/view/user_view.dart
class UserView extends StatelessWidget {
  const UserView({super.key});

  @override
  Widget build(BuildContext context) {
    return BlocBuilder<UserBloc, UserState>(
      builder: (context, state) {
        if (state is UserLoading) {
          return const CircularProgressIndicator();
        }
        if (state is UserLoaded) {
          return UserWidget(user: state.user);
        }
        return const SizedBox();
      },
    );
  }
}

// lib/src/ui/screens/pages/user/viewmodel/user_viewmodel.dart
class UserViewModel extends ChangeNotifier {
  final UserRepository _repository;
  
  UserViewModel(this._repository);
  
  Future<void> loadUser(String userId) async {
    // Lógica de negócio
  }
}

// lib/src/ui/screens/pages/user/view/widgets/user_widget.dart
class UserWidget extends StatelessWidget {
  final User user;
  
  const UserWidget({super.key, required this.user});
  
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text(user.name),
        Text(user.email),
      ],
    );
  }
}

// Widget reutilizável
// lib/src/ui/widgets/custom_button.dart
class CustomButton extends StatelessWidget {
  final String text;
  final VoidCallback onPressed;
  
  const CustomButton({
    super.key,
    required this.text,
    required this.onPressed,
  });
  
  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: onPressed,
      child: Text(text),
    );
  }
}
```

## 8. Configuração de Rotas

```dart
final router = GoRouter(
  initialLocation: '/',
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => const HomePage(),
    ),
    GoRoute(
      path: '/user/:id',
      builder: (context, state) => UserPage(
        userId: state.pathParameters['id']!,
      ),
    ),
  ],
);
```

## 9. Boas Práticas a Seguir

1. **Separação de Responsabilidades**:
   - Mantenha a lógica de negócios separada da UI
   - Use interfaces para abstração
   - Implemente injeção de dependência

2. **Testes**:
   - Escreva testes unitários para BLoCs
   - Teste casos de uso
   - Implemente testes de widget

3. **Documentação**:
   - Documente interfaces e classes públicas
   - Mantenha README atualizado
   - Use comentários para código complexo

4. **Performance**:
   - Use lazy loading
   - Implemente cache quando necessário
   - Otimize builds

5. **Manutenção**:
   - Siga os padrões de código
   - Mantenha a consistência
   - Faça revisões de código

## 10. Checklist de Inicialização

```markdown
[ ] Criar estrutura de pastas
[ ] Configurar pubspec.yaml
[ ] Implementar injeção de dependência
[ ] Configurar rotas
[ ] Implementar tema base
[ ] Configurar Firebase
[ ] Implementar repositório base
[ ] Configurar BLoC base
[ ] Implementar widgets base
[ ] Configurar internacionalização
```

## 11. Passos para Inicialização do Projeto

1. **Criar Novo Projeto**:

   ```bash
   flutter create meu_projeto
   cd meu_projeto
   ```

2. **Configurar Estrutura de Pastas**:

```bash
   # Estrutura base
   mkdir -p lib/src/{controllers,data,domain,infrastructure,ui}

   # Controllers
   mkdir -p lib/src/controllers/{bloc,cubit,services}

   # Data
   mkdir -p lib/src/data/{datasources,models,repositories}

   # Domain
   mkdir -p lib/src/domain/{entities,repositories,usecases}

   # Infrastructure
   mkdir -p lib/src/infrastructure/{config,services,utils,routes}

   # UI
   mkdir -p lib/src/ui/{screens,widgets,themes}
   mkdir -p lib/src/ui/screens/pages
   mkdir -p lib/src/ui/screens/pages/{view,viewmodel}
   mkdir -p lib/src/ui/screens/pages/view/widgets
   ```

3. **Configurar Dependências**:
   - Edite o arquivo `pubspec.yaml`
   - Execute `flutter pub get`

4. **Configurar Firebase**:
   - Siga as instruções do Firebase Console
   - Adicione o arquivo `firebase_options.dart`

5. **Implementar Configuração Base**:
   - Configure o `main.dart`
   - Implemente injeção de dependência
   - Configure rotas
   - Implemente tema base

6. **Desenvolver Primeira Feature**:
   - Crie entidades
   - Implemente repositório
   - Crie BLoC
   - Desenvolva UI

## 12. Recursos Adicionais

- [Documentação Flutter](https://flutter.dev/docs)
- [BLoC Pattern](https://bloclibrary.dev/)
- [GetIt](https://pub.dev/packages/get_it)
- [GoRouter](https://pub.dev/packages/go_router)
- [Firebase Flutter](https://firebase.flutter.dev/)

## 13. Metodologia de Escrita de Código e Padronizações

### 1. Padrão de Nomenclatura de Arquivos

1. **Controladores**:
   - Interface: `[nome]_controller.dart`
   - Implementação: `[nome]_controller_impl.dart`
   - Mock: `[nome]_controller_mock.dart`
   - Exemplo: `usuario_controller.dart`, `usuario_controller_impl.dart`

2. **Estados e Eventos BLoC**:
   - Estado: `[nome]_state.dart`
   - Evento: `[nome]_event.dart`
   - Exemplo: `financeiro_monitoramento_state.dart`

3. **Views e ViewModels**:
   - View: `[nome]_view.dart`
   - ViewModel: `[nome]_viewmodel.dart`
   - Exemplo: `autenticacao_view.dart`

### 2. Padrões de Implementação

1. **Interfaces e Implementações**:

   ```dart
   // Interface
   abstract class IUserRepository {
     Future<User> getUser(String userId);
   }

   // Implementação
   class UserRepository implements IUserRepository {
     @override
     Future<User> getUser(String userId) async {
       // Implementação
     }
   }
   ```

2. **Tratamento de Erros**:

   ```dart
   try {
     // Código
   } on Exception catch (e, stackTrace) {
     return Failure.fromError("Mensagem de erro", e, stackTrace);
   }
   ```

3. **Injeção de Dependência**:

   ```dart
   // Registro
   getIt.registerLazySingleton<UsuarioController>(() => UsuarioControllerImpl());

   // Uso
   final controller = getIt.get<UsuarioController>();
   ```

### 3. Padrões de Organização de Código

1. **Imports**:
   - Agrupe imports por origem (dart, flutter, pacotes, arquivos locais)
   - Use imports relativos para arquivos do mesmo módulo
   - Use imports absolutos para arquivos de outros módulos

2. **Classes**:
   - Mantenha classes pequenas e focadas
   - Use composição ao invés de herança
   - Implemente interfaces para abstração

3. **Métodos**:
   - Métodos devem ter uma única responsabilidade
   - Use nomes descritivos
   - Limite o número de parâmetros

### 4. Padrões de Documentação

1. **Comentários**:
   - Documente interfaces e classes públicas
   - Use comentários para explicar lógica complexa
   - Mantenha comentários atualizados

2. **Documentação de Métodos**:

   ```dart
   /// Obtém um usuário pelo ID
   /// [userId] - ID do usuário a ser obtido
   /// Retorna um [Future] com o [User] encontrado
   Future<User> getUser(String userId);
   ```

### 5. Padrões de Teste

1. **Testes Unitários**:
   - Teste cada componente isoladamente
   - Use mocks para dependências externas
   - Cubra casos de sucesso e erro

2. **Testes de Integração**:
   - Teste fluxos completos
   - Verifique integração entre componentes
   - Teste cenários reais

### 6. Padrões de Versionamento

1. **Commits**:
   - Use mensagens descritivas
   - Agrupe mudanças relacionadas
   - Siga o padrão: `tipo(escopo): descrição`

2. **Branches**:
   - `main` - Código de produção
   - `develop` - Desenvolvimento
   - `feature/*` - Novas funcionalidades
   - `bugfix/*` - Correções

### 7. Padrões de Performance

1. **Otimização**:
   - Use lazy loading
   - Implemente cache quando necessário
   - Evite reconstruções desnecessárias

2. **Memória**:
   - Libere recursos quando não utilizados
   - Use const quando possível
   - Evite vazamentos de memória

### 8. Padrões de Segurança

1. **Dados Sensíveis**:
   - Nunca armazene em texto plano
   - Use variáveis de ambiente
   - Implemente criptografia quando necessário

2. **Autenticação**:
   - Valide inputs
   - Implemente rate limiting
   - Use tokens seguros

### 9. Checklist de Qualidade de Código

```markdown
[ ] Seguir padrões de nomenclatura
[ ] Implementar interfaces
[ ] Tratar erros adequadamente
[ ] Documentar código
[ ] Escrever testes
[ ] Otimizar performance
[ ] Seguir padrões de segurança
[ ] Manter consistência
[ ] Revisar código
[ ] Refatorar quando necessário
