# C++17
### Nouveautés et fonctionnalités

- [Blocs d'initialisation `if`/`switch`](#if_init_statements)
- [maybe_unused](#maybe_unused)
- [nodiscard](#nodiscard)
- [fallthrough](#fallthrough)
- [Namespaces imbriqués](#nested_namespaces)
- [Fonction clamp](#clamp)
- [__has_include](#has_include)
- [Fold expressions](#fold_expressions)
- [Capture de `this` par valeur dans les lambdas](#this_capture)
- [Déduction de templates](#template_deduction)

---

#### Blocs d'initialisation `if`/`switch` <a id="if_init_statements"></a>

Désormais, il est possible de créer des variables dans les structures `if` tout comme c'est possible avec les `for`.

```cpp
if (const int i { 5 }; i < 10)
    std::cout << "Plus petit que 10" << std::endl;
else
    std::cout << "Plus grand que 10" << std::endl;    
```

Ici comme on peut le voir, plus besoin de créer la variable au dessus, elle est déclarée directement dans le `if`.
Cela permet d'éviter d'avoir à créer des variables dont la portée est supérieure à celle du `if` lorsqu'elles ne sont utilisées que dans la condition. De plus, lorsque l'on a plusieurs conditions à la suite, cela évite d'avoir à trouver différents noms pour les variables, ou bien d'avoir à alterner entre déclaration et réutilisation.

```cpp
std::vector<int> elements {1, 2, 3};

if (const auto it = std::find(elements.begin(), elements.end(), 2);
    it != elements.end())
{
    *it = 4;
}

if (const auto it = std::find(elements.begin(), elements.end(), 3);
    it != elements.end())
{
    *it = 7;
}
```

A noter que les variables créées dans un `if` ou `else if` sont accessibles dans tous les `else if` et `else` qui suivent.

```cpp
std::vector<int> elements {1, 2, 3};

if (const auto it = std::find(elements.begin(), elements.end(), 2); it != elements.end())
    *it = 4;
else
    elements.insert(it, 4);
    

// Équivalent à

{
    const auto it = std::find(elements.begin(), elements.end(), 2);
    if (it != elements.end())
        *it = 4;
    else
        elements.insert(it, 4);
}    
```

Cette structure d'initialisation est également disponible pour les `switch`.

```cpp
switch (const auto elements = str.split(';');
        elements.size())
{
    // ...
}
```

---

#### maybe_unused <a id="maybe_unused"></a>

En C++17, un nouvel attribut `[[maybe_unused]]` permet d'indiquer au compilateur qu'une entité comme une variable ou une fonction peut ne pas être utilisée dans le code, afin qu'il n'émette pas le warning correspondant. Ceci évite donc d'avoir à commenter le nom d'un paramètre comme on le faisait souvent par exemple lorsqu'une redéfinition n'utilise pas un paramètre de la méthode.

```cpp
// On évite le warning en commentant
void foo(int /*a*/)
{

}

// Utilisation de l'attribut
void foo([[maybe_unused]] int a)
{

}
```

On peut également utiliser l'attribut sur une variable ou encore une fonction.

```cpp
[[maybe_unused]]
int f(int a)
{
  return 2 * a;
}

void bar()
{
  [[maybe_unused]] int i = foo();
  assert(i == 2);
}
```

---

#### nodiscard <a id="nodiscard"></a>

L'attribut `[[nodiscard]]` lorsqu'il est utilisé sur une fonction, permet d'indiquer que le retour de la fonction ne doit pas être ignoré, mais récupéré ou utilisé. Dans le cas contraire, un warning est émis par le compilateur. Il peut être particulièrement intéressant pour forcer les tests de retour d'erreur des fonctions.

```cpp
[[nodiscard]]
int foo()
{
  return 2;
}

int main()
{
	foo();      // Warning : Valeur de retour ignorée
}
```

Il est également possible d'utiliser l'attribut avec des déclarations de type, commes les `struct`, `class` ou encore `enum`.

```cpp
class [[nodiscard]] A 
{
  
};

A foo()
{
  return {};
}

int main()
{
	foo();      // Warning : Valeur de retour ignorée
}
```

Dans ce cas, toute fonction retournant un objet du type déclaré avec `[[nodiscard]]` provoquera un warning si le retour est ignoré.

---

#### fallthrough <a id="fallthrough"></a>

Il est possible dans certains cas d'utilisation des `switch` de ne pas mettre de `break` pour que des actions soient communes à certains cas, comme dans l'exemple suivant.

```cpp
switch (a)
{
	case 1:
		std::cout << "1";
	case 2:
		std::cout << "2";
}
```

Dans le cas où le `case` en question n'est pas vide (comme le premier), le compilateur peut alors émettre un warning en pensant qu'il s'agit d'une erreur, car le cas 1 va alors afficher 1 et 2.

Si cela est volontaire, le C++17 permet d'utiliser l'attribut `[[fallthrough]]` afin de l'indiquer au compilateur.

```cpp
switch (a)
{
	case 1:
		std::cout << "1";
        [[fallthrough]];
	case 2:
		std::cout << "2";
}
```

---

#### Namespaces imbriqués <a id="nested_namespaces"></a>

Actuellement, lorsque l'on a plusieurs namespaces en C++, cela se traduit de la façon suivante.

```cpp
namespace first
{
	namespace second
	{
		class A
		{

		};
	}
}
```

Selon les conventions, les namespaces sont indentés ou non, mais dans tous les cas, leur indentation peut poser problème. Dans le premier cas, on obtient un niveau d'indentation élevé pour toute la classe ou tout le fichier dès lors que l'on a 3 ou 4 namespaces imbriqués, ce qui n'est pas très agréable. Dans l'autre cas, lorsqu'on choisit de ne pas indenter les espaces de noms, alors cela peut entrainer des conflits avec l'indentation automatique de certains éditeurs.

Afin de réduire ce problème, le C++17 autorise une nouvelle écriture des namespaces imbriqués, utilisant l'opérateur de portée `::`.

```cpp
namespace first::second
{
	class A
	{

	};
}
```

L'utilisation elle, ne change pas.

```cpp
first::second::A a;
```

---

#### Fonction clamp <a id="clamp"></a>

Il est parfois utilise de vouloir garder une valeur dans un intervalle, en lui imposant un minimum et un maximum, mais en gardant la valeur si elle est correcte. Pour cela, l'écriture la plus concise reste l'utilisation de ternaires.

```cpp
std::cout << (grade < 0 ? 0 : (grade > 20 ? 20 : grade)) << std::endl;
```

Pour cela, C++17 introduit une nouvelle fonction `clamp`.

```cpp
std::cout << std::clamp(grade, 0, 20) << std::endl;
```

---

#### __has_include <a id="has_include"></a>

Parfois, on peut avoir des inclusions qui se font sous certaines conditions. Les cas les plus fréquents sont les tests d'OS.

```cpp
#if defined(__linux__) || defined(__unix__) || defined(__APPLE__)
	#include <unistd.h>
#else
	#include <windows.h>
#endif
```

Une nouvelle directive `__has_include` permet de tester si le fichier que l'on veut inclure est disponible. Cela permet donc de s'adapter à l'environnement de manière simple.

```cpp
#if __has_include(<unistd.h>)
#include <unistd.h>
#endif

#if __has_include(<windows.h>)
#include <windows.h>
#endif
```

Un autre exemple d'utilisation peut concerner les futurs standards du langage. En effet, avant qu'une fonctionnalité soit standardisée, celle-ci est placée dans un espace de noms `experimental`. Ici, on peut donc aussi adapter l'inclusion en fonction afin que le code reste fonctionnel lors du changement.

```cpp
#if __has_include(<filesystem>)
	#include <filesystem>
#elif __has_include(<experimental/filesystem>)
	#include <experimental/filesystem>
```

---

#### Fold expressions <a id="fold_expressions"></a>

Il existe maintenant une nouvelle syntaxe pour l'utilisation des [templates variadiques](https://github.com/Manouel/Cxx-11/blob/master/README.md#variadic_templates) permettant d'éviter l'écriture récursive dans certains cas.

```cpp
// C++14
template<typename T>
T adder(T v)
{
	return v;
}

template<typename T, typename... Args>
auto adder(T first, Args... args)
{
	return first + adder(args...);
}


// C++17
template<typename... Args>
auto adder(Args... args)
{
	return (args + ...);
}
```

Ici l'expansion peut aussi bien se faire par la gauche que par la droite.

```cpp
template<typename... Args>
auto minus1(Args... args)
{
	return (args - ...);
}

template<typename... Args>
auto minus2(Args... args)
{
	return (... - args);
}
```

De la même manière, on peut ici calculer la moyenne des arguments.

```cpp
template<typename... Args>
auto avg(Args... args)
{
	return (args + ...) / sizeof... (args);
}
```

---

#### Capture de `this` par valeur dans les lambdas <a id="this_capture"></a>

En C++14, il n'est pas possible dans une lambda de capturer l'objet pointé par `this` par valeur, et l'accès à ses membres se fait donc toujours par référence. Cela peut poser certains problèmes si la lambda est distribuée de manière asynchrone, puisqu'il est alors possible lors de l'accès aux membres que le pointeur `this` ne soit plus valide alors que celui qui le manipule oui.
Il était tout de même possible d'en créer une copie soi-même :

```cpp
auto l = [=, tmp=*this](int i) {

};
```

Mais cela n'est pas toujours pratique et source d'erreur, puisque l'accès à un membre peut alors involontairement être faite au travers du pointeur `this`.

Le C++17 ajoute donc la possibilité de passer le pointeur vers l'objet courant par valeur.

```cpp
struct S
{
    int x;
    
    void f()
    {
        auto a = [this]() { x = 42 ; };  // (*this).x
        a();    // x = 42
        
        auto b = [*this]() { /*...*/ }; // this captured by value : read only access
    }
};
```

---

#### Déduction de templates <a id="template_deduction"></a>

Le C++17 introduit la déduction de templates. Là où avant on pouvait, pour certains types templates, utiliser une méthode de construction et `auto` pour utiliser la déduction de type, il est désormais possible de ne pas indiquer les arguments du template.

```cpp
// C++11/14
const auto t1 = std::make_tuple(3, 4);

// C++17
std::tuple t2(3,  4);
```

Cela permet donc de construire les objets suivants :

```cpp
std::array arr { 1, 2, 3 };     // std::array<int>
std::vector v { 2.2, 3.3 };     // std::vector<double>

std::pair p { "1", Foo() };     // std::pair<std::string, Foo>
std::tuple t { 1, 2, 'A' };     // std::tuple<int, int, char>
```

Il est également possible d'utiliser cette construction pour ses propres types template.

```cpp
template<class T>
struct A { A(T,T) {} };

A a(1, 2);                  // A<int>
auto ptrA = new A(1, 2);    // A<int>
```
