# C++17
### Nouveautés et fonctionnalités

- [maybe_unused](#maybe_unused)
- [nodiscard](#nodiscard)

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
