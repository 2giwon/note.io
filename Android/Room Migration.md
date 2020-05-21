> *SQLite API를 사용하여 데이터베이스 마이그레이션을 수행하면 항상 폭탄을 해체하는 것처럼 느껴졌습니다. 
마치 사용자의 손에 앱이 폭발하는 것을 잘못한 것처럼 보입니다.
Room을 사용하여 데이터베이스 작업을 처리하는 경우 스위치를 전환하는 것만 큼 쉽게 마이그레이션 할 수 있습니다.*

Room에서 데이터베이스 스키마를 변경했지만 버전을 업그레이드하지 않으면 앱이 중단됩니다. 

버전을 업그레이드했지만 마이그레이션을 제공하지 않으면 앱이 중단되거나 데이터베이스 테이블이 삭제되고 사용자가 데이터를 잃게됩니다.

마이그레이션을 구현하는 방법을 추측하여 (앱의) 생명을 위험에 빠뜨리지 마십시오.

오히려 Room이 내부에서 어떻게 작동하는지 이해하고 자신있게 데이터베이스를 마이그레이션하십시오.

## Database migrations under the hood

### What SQLite API does

SQLite 데이터베이스는 데이터베이스 버전 관리를 통해 스키마 변경을 처리합니다. 

보다 정확하게는 테이블을 추가, 제거 또는 수정하여 

스키마를 변경할 때마다 데이터베이스 버전 번호를 늘리고 `SQLiteOpenHelper.onUpgrade` 메소드 

구현을 업데이트해야합니다.

전 버전에서 새 버전으로 갈 때 수행해야 할 작업을 SQLite에 알려주는 방법입니다.

또한 앱이 데이터베이스 작업을 시작할 때 트리거되는 첫 번째 호출입니다. 

SQLite는 먼저 버전 업그레이드를 처리하려고 시도한 후에 만 데이터베이스를 엽니 다.

### What Room does

Room은 SQLite 마이그레이션을 `Migration` 클래스 형태로 

쉽게 수행 할 수있는 추상화 계층을 제공합니다. 

`Migration` 클래스는 특정 버전에서 

다른 버전으로 마이그레이션 할 때 수행해야 할 작업을 정의합니다. 

Room은 자체 `SQLiteOpenHelper` 구현을 사용하며 `onUpgrade` 메소드에서 

사용자가 정의한 마이그레이션을 트리거합니다.

*처음 데이터 베이스에 엑세스 하면 어떻게 됩니까?*

1. Room 데이터 베이스가 구축 됩니다.
2. `SQLiteOpenHelper.onUpgrade` 메소드가 호출되고 Room이 마이그레이션을 트리거합니다.
3. 데이터 베이스가 열립니다.

마이그레이션을 제공하지 않지만 

데이터베이스 버전을 늘리면 아래에서 고려해야 할 상황에 따라 

앱이 중단되거나 데이터가 손실 될 수 있습니다.

마이그레이션 내부에서 중요한 부분은 모든 데이터베이스 버전을 

고유하게 식별하기 위해 Room에서 사용하는 ID 해시 문자열에 의해 수행됩니다.

현재 버전의 ID 해시값은 데이터베이스의 Room에서 관리하는 구성 테이블에 보관됩니다.

따라서 데이터베이스를 들여다보고

 `room_master_table` 테이블을 보더라도 놀라지 마십시오.

---

두 개의 열이있는 `users` 테이블이 있는 간단한 예를 들어 보겠습니다.

- ID, `int`,  Primary key
- user name, `String`

`users` 테이블은 버전이 1 인 데이터베이스의 일부이며

`SQLiteDatabase` API를 사용하여 구현됩니다.

사용자가 이미 이 버전을 사용 중이고 

Room 사용을 시작하려고 한다고 가정하겠습니다. 

Room이 몇 가지 시나리오를 처리하는 방법을 살펴 보겠습니다.

## Migrate SQLite API code to Room

User Entity Class와 Data Access Class 인 `UserDao`가 만들어지고 

`RoomDatabase`를 확장하는 `UsersDatabase` 클래스에만 중점을 둔다고 가정 해 봅시다.

```java
@Database(entities = {User.class}, version = 1)
public abstract class UsersDatabase extends RoomDatabase
```

---

### Scenario 1: keep the database version unchanged — app crashes

데이터베이스 버전을 변경하지 않고 앱을 실행하는 경우 Room이 뒤에서 수행하는 작업은 다음과 같습니다.

**Step 1: Try to open the database**

현재 버전의 ID 해시와 `room_master_table`에 저장된 버전의 ID 해시를 비교하여 

데이터베이스의 ID를 확인 합니다. 

그러나 ID 해시가 저장되지 않았으므로 앱이 `IllegalStateException`이 발생합니다. ❌

```java
*java.lang.IllegalStateException: Room cannot verify the data integrity. 
Looks like you’ve changed schema but forgot to update the version number. 
You can simply fix this by increasing the version number.*
```

> *데이터베이스 스키마를 수정하지만 버전 번호를 업데이트하지 않으면 Room에서 항상 `IllegalStateException`이 발생합니다.*

데이터 베이스 버전을 올립니다.

```java
@Database(entities = {User.class}, **version = 2**)
public abstract class UsersDatabase extends RoomDatabase
```

---

### Scenario 2: version increased, but no migration provided — app crashes

이제 앱을 다시 실행할 때 Room은 다음을 수행합니다.

Step 1 : 기기에 설치된 버전 1에서 버전 2로 업그레이드

- 마이그레이션이 없으므로 응용 프로그램이 `IllegalStateException`과 충돌합니다.❌

```java
*java.lang.IllegalStateException: A migration from 1 to 2 is necessary. 
Please provide a Migration in the builder or call fallbackToDestructiveMigration 
in the builder in which case Room will re-create all of the tables.*
```

> *마이그레이션을 제공하지 않으면 Room에서 `IllegalStateException`이 발생합니다.*

---

### Scenario 3: version increased, fallback to destructive migration enabled — database is cleared

마이그레이션을 제공하지 않고 버전을 업그레이드 할 때 

데이터베이스를 명확하게 지우려면 

데이터베이스 빌더에서 `fallbackToDestructiveMigration`을 

호출하십시오.

```java
Room.databaseBuilder(context.getApplicationContext(), UsersDatabase.class, "Sample.db")
							.fallbackToDestructiveMigration()
							.build();
```

이제 앱을 다시 실행할 때 Room은 다음을 수행합니다.

Step 1 : 기기에 설치된 버전 1에서 버전 2로 업그레이드

- 마이그레이션이없고 `fallbackToDestructive Migration`으로 대체되므로 테이블이 삭제되고 identity_hash가 삽입됩니다. 🤷

Step 2: 데이터베이스를 엽니다.

- 현재 버전의 ID 해시와 `room_master_table`에 저장된 것과 동일합니다. ✅

이제 앱이 다운되지 않지만 모든 데이터가 손실됩니다. 

따라서 이것이 마이그레이션을 구체적으로 처리하려는 방법인지 확인하십시오.

---

### Scenario 4: version increased, migration provided — data is kept

사용자의 데이터를 유지하려면 마이그레이션을 구현해야합니다. 

스키마는 변경되지 않으므로 Empty Migration Implement 를 제공하고 

Room에 적용하면 됩니다.

```java
@Database(entities = {User.class}, version = 2)
public abstract class UsersDatabase extends RoomDatabase {

…

static final Migration MIGRATION_1_2 = new Migration(1, 2) {
    @Override
    public void migrate(SupportSQLiteDatabase database) {
        // Since we didn't alter the table, there's nothing else to do here.
    }
};

…

database =  Room.databaseBuilder(context.getApplicationContext(),
        UsersDatabase.class, "Sample.db")
        .addMigrations(MIGRATION_1_2)
        .build();
```

앱을 실행할 때 Room은 다음을 수행합니다.

Step 1. Device에 설치된 버전 1을 버전 2로 업그레이드하십시오.

- 빈 마이그레이션 트리거 ✅
- `room_master_table`에서 ID 해시 업데이트 ✅

Step 2: 데이터 베이스를 엽니다.

- 현재 버전의 ID 해시와 `room_master_table`에 저장된 것과 동일합니다. ✅

이제 앱이 열리고 사용자 데이터가 마이그레이션됩니다! 🎉

## Migration with simple schema changes

`User` 클래스를 수정하여 다른 열인 `last_update`를 users 테이블에 추가하겠습니다.

`UsersDatabase` 클래스에서 다음 변경을 수행해야합니다.

1. 버전을 3으로 증가 시킵니다.

    ```java
    @Database(entities = {User.class}, **version = 3**)
    public abstract class UsersDatabase extends RoomDatabase
    ```

2. 버전 2에서 버전 3으로 마이그레이션 추가합니다.

    ```java
    static final Migration MIGRATION_2_3 = new Migration(2, 3) {
        @Override
        public void migrate(SupportSQLiteDatabase database) {
            database.execSQL("ALTER TABLE users "
                    + " ADD COLUMN last_update INTEGER");
        }
    };
    ```

3. Room DataBase Builder에 마이그레이션을 추가하십시오.

    ```java
    database = Room.databaseBuilder(context.getApplicationContext(),
            UsersDatabase.class, "Sample.db")
            ***.addMigrations(MIGRATION_1_2, MIGRATION_2_3)***
            .build();
    ```

앱을 실행 하면 다음 단계가 수행 됩니다:

Step 1: 기기에 설치된 버전 2에서 버전 3으로 업그레이드

- 사용자의 데이터를 유지하면서 마이그레이션을 트리거하고 테이블을 변경합니다. ✅
- `room_master_table`  안의 ID 해시를 업데이트 합니다. ✅

Step 2: 데이터 베이스를 엽니다.

- 현재 버전의 ID 해시와 `room_master_table`에 저장된 것과 동일합니다.✅

## Migrations with complex schema changes

SQLite의 `ALTER TABLE…` 명령은 상당히 제한적입니다. 예를 들어, 사용자 ID를 int에서 String으로 변경하려면 몇 가지 단계가 필요합니다.

- 새로운 스키마로 새로운 임시 테이블을 만들고
- `users` 테이블에서 임시 테이블로 데이터를 복사하고
- `users` 테이블을 삭제
- 임시 테이블 이름을 `users`로 바꿉니다.

Room을 사용하여 마이그레이션 구현은 다음과 같습니다.

```java
static final Migration MIGRATION_3_4 = new Migration(3, 4) {
    @Override
    public void migrate(SupportSQLiteDatabase database) {
        // Create the new table
        database.execSQL(
                "CREATE TABLE users_new (userid TEXT, username TEXT, last_update INTEGER, PRIMARY KEY(userid))");
				// Copy the data
        database.execSQL(
                "INSERT INTO users_new (userid, username, last_update) SELECT userid, username, last_update FROM users");
				// Remove the old table
        database.execSQL("DROP TABLE users");
				// Change the table name to the correct one
        database.execSQL("ALTER TABLE users_new RENAME TO users");
    }
};
```

## Multiple database version increments

사용자에게 데이터베이스 버전 1을 실행중인 이전 버전의 앱이 있고 

버전 4로 업그레이드하려면 어떻게해야합니까? 

지금까지 버전 1에서 2로, 

버전 2에서 3으로, 

버전 3에서 4 로의 마이그레이션을 정의 했으므로 

Room은 모든 마이그레이션을 차례로 트리거합니다.

Room은 둘 이상의 버전 증분을 처리 할 수 있습니다. 

한 단계로 버전 1에서 4로 이동하는 마이그레이션을 정의하여 

마이그레이션 프로세스를 더 빠르게 수행 할 수 있습니다.

```java
static final Migration MIGRATION_1_4 = new Migration(1, 4) {
    @Override
    public void migrate(SupportSQLiteDatabase database) {
        // Create the new table
        database.execSQL(
                "CREATE TABLE users_new (userid TEXT, username TEXT, last_update INTEGER, PRIMARY KEY(userid))");
        // Copy the data
        database.execSQL(
                "INSERT INTO users_new (userid, username, last_update) SELECT userid, username, last_update FROM users");
				// Remove the old table
        database.execSQL("DROP TABLE users");
				// Change the table name to the correct one
        database.execSQL("ALTER TABLE users_new RENAME TO users");
    }
};
```

다음으로 마이그레이션 목록에 추가합니다.

```java
database = Room.databaseBuilder(context.getApplicationContext(),
        UsersDatabase.class, "Sample.db")
        .addMigrations(MIGRATION_1_2, MIGRATION_2_3, MIGRATION_3_4, MIGRATION_1_4)
        .build();
```

> *Migration.migrate() 구현에서 작성한 쿼리는 `DAO`의 쿼리와 달리 런타임에 컴파일되지 않습니다. 마이그레이션 테스트를 구현하고 있는지 확인하십시오.*
