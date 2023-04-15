# Sqlite-DB
Sqlite orm modelled after django orm

# Quick Demo
```python
from sqlite import SqliteDatabase

db = SqliteDatabase("./db.sqlite")

@db.on("error")
def _(err):
    raise err

class Session(db.Model):
    sid: int
    data: str

class User(db.Model):
    name: str
    age: int
    password: bytes

    def verify_password(self, password):
        return verify_password(
            self.name, password, self.password
        )


class Note(db.Model):
    name: str
    body: str = '' # Default field
    user: User # Foreign Key assignment
    users: list[User] # Many to one keys

```

Or for more control on how the models are created
Authentication and authorization model demo
```python
from sqlite import SqliteDatabase
from sqlite.columns import *
from sqlite.files import FileSystemStorage

database = SqliteDatabase("./db.sqlite")

@database.on('error')
def error(e):
    raise e

# For easier access to files and folders
# Call create to create directory if not exists
store = FileSystemStorage(directory="./media").create()

# Go to subdirectory
# Sames api as FileSystemStorage object
uploads = store["uploads"].create()

class Role(database.Model):
    role: str = SelectColumn(options=["admin", "editor", "user", "superuser"])
    level: int = IntegerColumn(null=False, min=1, max=100)

    def __str__(self):
        return f"[{self.id}] role -> {self.role}: level -> {self.level}"

    class Meta:
        table_name = "auth_role"

class User(database.Model):
    username: str
    email_addr: str = EmailColumn(label="Email Address", helper_text="User's email address: '@' must be present", unique=True)
    desc: str = RichTextColumn(label="Description", helper_text="Description of the user")
    creation_date: datetime = DateTimeColumn(null=False)
    image: str = ImageColumn(extensions=["jpg", "png"], store=uploads, default="default.png", label="Image")
    last_login: datetime = DateTimeColumn()
    hash: bytes = BytesColumn(null=False)
    is_active: bool = BooleanColumn()

    @property
    def tag(self):
        return f"@{self.username}".lower()
    
    def __str__(self):
        return f"[{self.id}] username -> {self.username}"
    
    class Meta:
        table_name = "auth_users"

class PendingReg(database.Model):
    code: str
    email_addr: str = EmailColumn(label="Email Address")
    desc: str = Column(editor="full", label="Description")
    hash: bytes = BytesColumn(null=False)
    username: str = Column(null=False)
    creation_date: datetime = DateTimeColumn(null=False, label="Date Created")
    
    class Meta:
        table_name = "auth_pending_registrations"

class Token(database.Model):
    tokenid: str
    last_used: datetime = DateTimeColumn(null=True)
    is_expired = BooleanColumn(default=False)
    usage_amount: int = IntegerColumn(default=0)
    usage_limit: int = IntegerColumn()
    user = ForeignKeyColumn(User)
    value: str = Column(null=False)
    creation_date: datetime = DateTimeColumn(null=False, label="Created")
    expiry_date: datetime = DateTimeColumn(null=True)
    
    @property
    def owner(self):
        return User.objects.get(id=self.user_id)
    
    class Meta:
        table_name = "auth_tokens"

class AuthAttempts(database.Model):
    ip_address: str = Column(null=False)
    user_agent: str = Column(null=False)
    token: str = Column(null=False)
    created_at = DateTimeColumn()

    class Meta:
        table_name = "auth_activation_attempts"

class Permissions(database.Model):
    name: str = Column(null=False)
    description: str = Column(null=False)

    class Meta:
        table_name = 'auth_permissions'

class UsersPermissions(database.Model):
    user = ForeignKeyColumn(User)
    permission = ForeignKeyColumn(Permissions)

    class Meta:
        table_name = 'auth_users_permissions'

class Groups(database.Model):
    name: str = Column(null=False)
    description: str = Column(null=False)

    class Meta:
        table_name = 'auth_groups'

class GroupsUsers(database.Model):
    user: int = ForeignKeyColumn(User)
    group: int = ForeignKeyColumn(Groups, name="user_group")

    class Meta:
        table_name = 'auth_groups_users'

class GroupsPermissions(database.Model):
    group = ForeignKeyColumn(Groups, name="user_group")
    permission = ForeignKeyColumn(Permissions)

    class Meta:
        table_name = 'auth_groups_permissions'
```
