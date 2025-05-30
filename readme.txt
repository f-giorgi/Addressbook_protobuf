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




