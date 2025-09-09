# LocalStorage

## Save data
localStorage.setItem("key", "value");

## Get data
localStorage.getItem("key");

## Remove data
localStorage.removeItem("key");

## Clear all data
localStorage.clear();


# Exemples
```
localStorage.setItem("user", JSON.stringify({ name: "Vincent", age: 30 }));
const user = JSON.parse(localStorage.getItem("user")); // → output: { name: "Vincent", age: 30 };
```
