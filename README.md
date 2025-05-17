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

### `ItemDatabase.kt`

```kotlin
@Database(entities = [ItemModel::class], version = 1)
abstract class ItemDatabase : RoomDatabase() {
    abstract fun itemDao(): ItemDao
}
```

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
