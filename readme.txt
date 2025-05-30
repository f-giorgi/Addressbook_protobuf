##### 1
# BUILD e INSTALLAZIONE dell'ambiente protobuf (compilatore e librerie) su ubuntu linux 20.04 LTS
# In questo modo sono allineati alla stessa versione.

# Richiede cmake e ninja installati oltre al g++
workspace> sudo apt update && sudo apt upgrade -y && sudo apt install -y cmake ninja-build

# clona la repository
workspace> git clone https://github.com/protocolbuffers/protobuf.git && cd protobuf

# porta i sorgenti ad una tag stabile
protobuf> git checkout v25.8

# inizializza i submodule (scarica le dipendenze come abseil)
protobuf> git submodule update --init --recursive

# crea cartella di lavoro
protobuf> mkdir build && cd build

# configura compilazione 
build> cmake .. -G Ninja -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr/local -Dprotobuf_BUILD_TESTS=OFF

# compilazione parallela su tutti i core
build> cmake --build . --parallel $(nproc)

# installa in /usr/local (include, lib ecc)
build> sudo cmake --install .

#### 2
# BUILD dell'esempio addressbook preso da https://protobuf.dev/getting-started/cpptutorial/
# e' comodo aprire la directory che contiene questo readme.txt direttamente con vscode se e' 
# ha i plugin 'cmake-tools' e 'C/C++ Extension Pack'.
# In alternativa si puo' compilare il progetto da riga di comando in modo simile a come
# e' stato compilato l'ambiente protobuf.

# crea cartella di lavoro
Addressbook_protobuf> mkdir build && cd build

# configura build
build> cmake -G Ninja ../

# compila esempio
build> cmake --build .


###### NOTE
In Protobuf (Protocol Buffers), **non esiste un supporto diretto per l’ereditarietà** 
come in C++ o in altri linguaggi orientati agli oggetti. 
Tuttavia, ci sono alcune **strategie comuni per simulare o gestire l'ereditarietà** 
nei messaggi `.proto`.

---

## 🚫 Cosa *non* si può fare

Non puoi scrivere in `.proto` qualcosa del tipo:

```proto
message Base {
  int32 id = 1;
}

message Derived : Base { // ❌ ERRORE: Protobuf non supporta l’ereditarietà
  string name = 2;
}
```

---

## ✅ Soluzioni alternative (Design Pattern)

### 1. **Composizione**

Usa un messaggio dentro l’altro, anziché ereditarietà.

```proto
message Base {
  int32 id = 1;
}

message Derived {
  Base base = 1;
  string name = 2;
}
```

In C++ genererà qualcosa come:

```cpp
Derived d;
d.mutable_base()->set_id(42);
d.set_name("Mario");
```

---

### 2. **Oneof per simulare polimorfismo**

Puoi definire un messaggio contenitore che può contenere un "tipo base" con più opzioni.

```proto
message Animal {
  oneof kind {
    Dog dog = 1;
    Cat cat = 2;
  }
}

message Dog {
  string breed = 1;
}

message Cat {
  int32 lives = 1;
}
```

In C++ puoi poi fare:

```cpp
Animal a;
a.mutable_dog()->set_breed("Labrador");
```

E verificare il tipo con `a.has_dog()` o `a.has_cat()`.

---

### 3. **Discriminator manuale + campi opzionali**

Puoi usare un campo tipo enum per distinguere il tipo e poi avere campi opzionali.

```proto
enum ShapeType {
  SHAPE_UNKNOWN = 0;
  SHAPE_CIRCLE = 1;
  SHAPE_RECTANGLE = 2;
}

message Shape {
  ShapeType type = 1;

  Circle circle = 2;
  Rectangle rectangle = 3;
}

message Circle {
  double radius = 1;
}

message Rectangle {
  double width = 1;
  double height = 2;
}
```

Questo è simile a `oneof` ma ti consente più flessibilità (anche se meno controllo sulla coerenza).

---

## 🧠 Considerazioni

* **Protobuf è pensato per la serializzazione**, non per il modeling OO complesso.
* Se stai lavorando in C++ e vuoi sfruttare il polimorfismo a runtime, puoi fare un **mapping tra tipi Protobuf e classi C++** polimorfe, ma questo richiede un adattatore esterno (es. una `Factory` che costruisce oggetti C++ dai messaggi Protobuf).

---

## ✅ Conclusione

No, **non puoi usare l’ereditarietà diretta in Protobuf**, ma puoi **simularla efficacemente** con:

* `oneof` per polimorfismo,
* composizione per riuso di campi,
* enum + campi opzionali per maggiore controllo.

Se mi descrivi il tuo caso specifico, posso suggerirti lo schema `.proto` e l’interfaccia C++ più adatti.
