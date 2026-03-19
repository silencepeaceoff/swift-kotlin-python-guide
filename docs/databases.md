# 🗄️ Базы данных

=== "Swift 6"
    ```swift
    // SwiftData (Swift 5.9+, macOS 14+, iOS 17+) — современный ORM от Apple
    import SwiftData

    @Model
    final class Article {
        var id: UUID = UUID()
        var title: String
        var body: String
        var createdAt: Date = Date()
        @Relationship(deleteRule: .cascade) var tags: [Tag] = []

        init(title: String, body: String) {
            self.title = title
            self.body = body
        }
    }

    @Model
    final class Tag {
        var name: String
        init(name: String) { self.name = name }
    }

    // Настройка контейнера
    let container = try ModelContainer(
        for: Article.self, Tag.self,
        configurations: ModelConfiguration(isStoredInMemoryOnly: false)
    )
    let context = container.mainContext

    // CRUD
    let article = Article(title: "Hello", body: "World")
    context.insert(article)
    try context.save()

    // Fetch с предикатом
    let descriptor = FetchDescriptor<Article>(
        predicate: #Predicate { $0.title.contains("Hello") },
        sortBy: [SortDescriptor(\.createdAt, order: .reverse)]
    )
    let articles = try context.fetch(descriptor)

    // Удаление
    context.delete(article)

    // В SwiftUI:
    // @Query(sort: \.createdAt) var articles: [Article]
    ```

=== "Kotlin 2.x"
    ```kotlin
    // ── Room (Android) ──
    import androidx.room.*

    @Entity(tableName = "articles")
    data class Article(
        @PrimaryKey(autoGenerate = true) val id: Long = 0,
        @ColumnInfo(name = "title") val title: String,
        val body: String,
        val createdAt: Long = System.currentTimeMillis()
    )

    @Dao
    interface ArticleDao {
        @Query("SELECT * FROM articles ORDER BY createdAt DESC")
        fun getAllFlow(): Flow<List<Article>>       // реактивное чтение!

        @Query("SELECT * FROM articles WHERE title LIKE '%' || :query || '%'")
        suspend fun search(query: String): List<Article>

        @Insert(onConflict = OnConflictStrategy.REPLACE)
        suspend fun insert(article: Article): Long

        @Update
        suspend fun update(article: Article)

        @Delete
        suspend fun delete(article: Article)
    }

    @Database(entities = [Article::class], version = 1, exportSchema = true)
    abstract class AppDatabase : RoomDatabase() {
        abstract fun articleDao(): ArticleDao
    }

    // Инициализация (singleton)
    val db = Room.databaseBuilder(context, AppDatabase::class.java, "app.db")
        .fallbackToDestructiveMigration()
        .build()

    val dao = db.articleDao()
    dao.insert(Article(title = "Hello", body = "World"))
    dao.getAllFlow().collect { articles -> updateUI(articles) }

    // ── Exposed (JVM / серверный) ──
    import org.jetbrains.exposed.sql.*
    import org.jetbrains.exposed.sql.transactions.transaction

    object Articles : Table("articles") {
        val id    = integer("id").autoIncrement()
        val title = varchar("title", 255)
        val body  = text("body")
        override val primaryKey = PrimaryKey(id)
    }

    Database.connect("jdbc:postgresql://localhost/mydb", "org.postgresql.Driver",
                     user = "user", password = "pass")

    transaction {
        SchemaUtils.create(Articles)
        Articles.insert { it[title] = "Hello"; it[body] = "World" }
        Articles.selectAll().where { Articles.title like "%Hello%" }
               .map { it[Articles.title] }
    }
    ```

=== "Python 3.13"
    ```python
    # SQLAlchemy 2.x (ORM стандарт) + SQLite (встроен)
    from sqlalchemy import String, Integer, DateTime, ForeignKey, select, func
    from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column, relationship
    from sqlalchemy.ext.asyncio import AsyncSession, create_async_engine, async_sessionmaker
    from datetime import datetime, UTC

    class Base(DeclarativeBase):
        pass

    class Article(Base):
        __tablename__ = "articles"
        id:         Mapped[int]      = mapped_column(Integer, primary_key=True, autoincrement=True)
        title:      Mapped[str]      = mapped_column(String(255), nullable=False, index=True)
        body:       Mapped[str]      = mapped_column(String, nullable=False)
        created_at: Mapped[datetime] = mapped_column(DateTime(timezone=True),
                                                     default=lambda: datetime.now(UTC))
        tags:       Mapped[list["Tag"]] = relationship("Tag", back_populates="article",
                                                        cascade="all, delete-orphan")

    class Tag(Base):
        __tablename__ = "tags"
        id:         Mapped[int] = mapped_column(primary_key=True)
        name:       Mapped[str] = mapped_column(String(50))
        article_id: Mapped[int] = mapped_column(ForeignKey("articles.id"))
        article:    Mapped[Article] = relationship("Article", back_populates="tags")

    # Async Engine + Session
    engine = create_async_engine("postgresql+asyncpg://user:pass@localhost/mydb",
                                 pool_size=10, max_overflow=20)
    AsyncSessionLocal = async_sessionmaker(engine, expire_on_commit=False)

    # CRUD
    async def create_article(title: str, body: str) -> Article:
        async with AsyncSessionLocal() as session:
            article = Article(title=title, body=body)
            session.add(article)
            await session.commit()
            await session.refresh(article)
            return article

    async def get_articles(search: str | None = None) -> list[Article]:
        async with AsyncSessionLocal() as session:
            stmt = select(Article).order_by(Article.created_at.desc())
            if search:
                stmt = stmt.where(Article.title.ilike(f"%{search}%"))
            result = await session.execute(stmt)
            return list(result.scalars().all())

    # SQLite (встроен, без установки)
    import sqlite3
    with sqlite3.connect("local.db") as conn:
        conn.execute("CREATE TABLE IF NOT EXISTS kv (key TEXT PRIMARY KEY, value TEXT)")
        conn.execute("INSERT OR REPLACE INTO kv VALUES (?, ?)", ("name", "Alice"))
        row = conn.execute("SELECT value FROM kv WHERE key = ?", ("name",)).fetchone()
        print(row[0])                              # "Alice"
    ```
