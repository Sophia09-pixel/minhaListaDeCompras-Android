# 📱 Lista de Compras - Aplicativo Android

Este projeto é um exemplo de aplicativo Android que permite criar e remover itens de uma lista de compras. Ele utiliza os principais componentes modernos do Android como:

- **RecyclerView** para listagem dinâmica
- **Room Database** para persistência local
- **ViewModel** para manter os dados mesmo com mudanças de configuração
- **Coroutines** para operações assíncronas

---
antes de adicionar um item

![image](https://github.com/user-attachments/assets/1190354c-8172-49c2-b02d-9ba4f6829d6c)


depois de adicionar

![image](https://github.com/user-attachments/assets/a1e8778b-f144-408f-8138-773df5348594)


depois de apagar o item cafe

![image](https://github.com/user-attachments/assets/77c6213d-0874-4b33-b10b-61c0961d3c6b)

---

## 🧱 Estrutura do Projeto

```
├── model/
│   └── ItemModel.kt
├── data/
│   ├── ItemDao.kt
│   └── ItemDatabase.kt
├── viewmodel/
│   └── ItemsViewModel.kt
├── adapter/
│   └── ItemsAdapter.kt
├── MainActivity.kt
├── res/
│   ├── layout/
│   │   ├── activity_main.xml
│   │   └── item_layout.xml
│   └── drawable/
│       └── ic_delete.xml
└── AndroidManifest.xml
```

---

## 🧩 Conceitos Envolvidos

### 🔁 RecyclerView

- Estrutura para exibir listas de forma eficiente e reaproveitável.
- Cada item é um objeto (`ItemModel`) com seu próprio layout.
- Quando a lista é rolada, o RecyclerView reaproveita os elementos da tela, melhorando a performance.

### 💾 Room Database

- Abstração moderna sobre o SQLite.
- Cria tabelas a partir de `@Entity` e permite acesso via `@Dao`.

### 🧠 ViewModel + LiveData

- Separa lógica de UI dos dados.
- LiveData permite observar alterações nos dados.
- ViewModel sobrevive a mudanças de configuração (como rotação de tela).

### 🧵 Coroutines

- Operações em background (acesso ao banco, por exemplo) são feitas com `Dispatchers.IO`.
- Evita travamento da interface ao manipular dados.

---

## 🛠️ Dependências (build.gradle)

```kotlin
implementation("androidx.room:room-ktx:2.4.1")
kapt("androidx.room:room-compiler:2.4.1")

implementation("androidx.lifecycle:lifecycle-livedata-ktx:2.3.1")
implementation("androidx.appcompat:appcompat:1.4.1")
implementation("androidx.activity:activity-ktx:1.7.0")
implementation("org.jetbrains.kotlinx:kotlinx-coroutines-android:1.5.2")
```
### implementation("androidx.room:room-ktx:2.4.1"):

- Room KTX é uma extensão do Room, que é a biblioteca de persistência de dados do Android. O Room KTX fornece APIs Kotlin-friendly para facilitar a integração com Room, utilizando recursos como funções de extensão (por exemplo, suspend functions para operações assíncronas). Essa dependência é útil para simplificar o uso de Room em projetos Kotlin.

### kapt("androidx.room:room-compiler:2.4.1"):

- Room Compiler é necessário para gerar código automaticamente para as classes que definem as entidades e os DAO (Data Access Object). O Room usa processamento de anotação para gerar esse código, e o KAPT (Kotlin Annotation Processing Tool) é necessário para que o Kotlin execute esse processo. Portanto, essa dependência é fundamental para compilar os componentes do Room em um projeto Kotlin.

### implementation("androidx.lifecycle:lifecycle-livedata-ktx:2.3.1"):

- LiveData KTX é uma extensão do LiveData, que é uma classe do Android Lifecycle que permite gerenciar dados observáveis. O LiveData KTX facilita o uso do LiveData em Kotlin, com funções de extensão que tornam o código mais conciso e expressivo. Ele é usado para observar dados que podem ser alterados ao longo do tempo e garantir que a interface do usuário seja atualizada de forma reativa.

### implementation("androidx.appcompat:appcompat:1.4.1"):

- AppCompat é uma biblioteca que fornece compatibilidade com versões antigas do Android, permitindo usar os recursos mais recentes em versões do Android que não os suportam nativamente. Isso inclui, por exemplo, temas, widgets e recursos de UI modernos em dispositivos mais antigos. A dependência appcompat garante que sua aplicação tenha uma aparência e comportamento consistentes em diferentes versões do Android.

### implementation("androidx.activity:activity-ktx:1.7.0"):

- Activity KTX fornece extensões Kotlin para trabalhar com Activity no Android. Essas extensões tornam o código mais conciso e fácil de ler, simplificando tarefas comuns ao interagir com a Activity, como navegação, manipulação de resultados de atividades e outros aspectos do ciclo de vida da Activity.

### implementation("org.jetbrains.kotlinx:kotlinx-coroutines-android:1.5.2"):

- Kotlin Coroutines for Android é a dependência necessária para usar coroutines no Android. As coroutines são uma maneira eficiente de realizar operações assíncronas no Kotlin, como chamadas de rede, manipulação de IO ou tarefas longas, sem bloquear a thread principal. Essa dependência fornece suporte específico para o uso de coroutines em aplicativos Android.

---

## 📦 Banco de Dados

### `ItemModel.kt`

```kotlin
@Entity
data class ItemModel(
    @PrimaryKey(autoGenerate = true)
    val id: Int = 0,
    val name: String
)
```
- Define a entidade do banco de dados.

- @Entity: marca a classe como uma tabela no banco de dados Room.

- @PrimaryKey(autoGenerate = true): define o campo id como chave primária gerada automaticamente.

- val name: String: campo da tabela que armazena o nome do item.

### `ItemDao.kt`

```kotlin
@Dao
interface ItemDao {
    @Query("SELECT * FROM ItemModel")
    fun getAll(): LiveData<List<ItemModel>>

    @Insert
    fun insert(item: ItemModel)

    @Delete
    fun delete(item: ItemModel)
}
```
- Define o Data Access Object (DAO) com as operações no banco:

- getAll(): retorna todos os itens como LiveData, permitindo atualização automática da interface quando os dados mudarem.

- insert(item): insere um novo item na tabela.

- delete(item): remove um item da tabela.

### `ItemDatabase.kt`

```kotlin
@Database(entities = [ItemModel::class], version = 1)
abstract class ItemDatabase : RoomDatabase() {
    abstract fun itemDao(): ItemDao
}
```
- Define o banco de dados Room:

- @Database(...): especifica as entidades (tabelas) e a versão.

- abstract fun itemDao(): função para obter a instância do DAO.

---

## 🧠 ViewModel

### `ItemsViewModel.kt`

```kotlin
class ItemsViewModel(application: Application) : AndroidViewModel(application) {

    private val itemDao: ItemDao

    init {
        val database = Room.databaseBuilder(
            application,
            ItemDatabase::class.java,
            "items_database"
        ).build()

        itemDao = database.itemDao()
    }

    fun addItem(item: String) {
        viewModelScope.launch(Dispatchers.IO) {
            val newItem = ItemModel(name = item)
            itemDao.insert(newItem)
        }
    }

    fun removeItem(item: ItemModel) {
        viewModelScope.launch(Dispatchers.IO) {
            itemDao.delete(item)
        }
    }
}
```
- Classe ViewModel que gerencia os dados da UI de forma reativa e sobrevive a mudanças de configuração (como rotação de tela).

- Room.databaseBuilder(...): cria a instância do banco de dados Room.

- viewModelScope.launch(Dispatchers.IO): executa as operações de banco em uma thread separada (não na principal).

- addItem: cria e insere um novo item.

- removeItem: remove o item informado.

---

## 🔄 ItemsAdapter

```kotlin
class ItemsAdapter(private val onItemRemoved: (ItemModel) -> Unit)
    : RecyclerView.Adapter<ItemsAdapter.ItemViewHolder>() {

    private var items = listOf<ItemModel>()

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ItemViewHolder {
        val view = LayoutInflater.from(parent.context)
            .inflate(R.layout.item_layout, parent, false)
        return ItemViewHolder(view)
    }

    override fun getItemCount(): Int = items.size

    override fun onBindViewHolder(holder: ItemViewHolder, position: Int) {
        val item = items[position]
        holder.bind(item)
    }

    inner class ItemViewHolder(view: View) : RecyclerView.ViewHolder(view) {
        private val textView: TextView = view.findViewById(R.id.textView)
        private val button: Button = view.findViewById(R.id.deleteButton)

        fun bind(item: ItemModel) {
            textView.text = item.name
            button.setOnClickListener {
                onItemRemoved(item)
            }
        }
    }
}
```
- Adapta os dados (ItemModel) para exibição em uma lista (RecyclerView).

- onItemRemoved: função passada como callback para remover o item ao clicar no botão.

- Lista de itens exibida no RecyclerView.

- Cria a visualização de cada item com base no item_layout.xml.

- Liga os dados de um item (ItemModel) ao seu ViewHolder.

- Classe interna que representa a visualização de um item.

- Mostra o nome (item.name) e define o clique do botão para remover o item via callback.

---

## 🖼️ Interface XML

- `activity_main.xml`: Contém o campo de texto (EditText), botão de adicionar e RecyclerView.
- `item_layout.xml`: Layout de cada item com texto e botão de exclusão.

---

## 🧪 Inspecionar o Banco

1. Acesse `View > Tool Windows > App Inspection`
2. Vá em:
```
Device Explorer > data > data > pacote_do_app > database > items_database
```
3. Visualize e edite os dados diretamente.

---

## ✅ Observações Extras

- Se o botão de adicionar for clicado com o campo vazio, um erro aparece no EditText.
- O `RecyclerView` calcula automaticamente quantos itens cabem na tela.
- O botão de remover chama `onItemRemoved`, que por sua vez chama `removeItem()` no ViewModel.
