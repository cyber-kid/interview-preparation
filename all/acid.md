# ACID Consistency Model
The key ACID guarantee is that it provides a safe environment in which to operate on your data. The ACID acronym stands for:

1. **Atomic** All operations in a transaction succeed or every operation is rolled back.
2. **Consistent** On the completion of a transaction, the database is structurally sound.
3. **Isolated** Transactions do not contend with one another. Contentious access to data is moderated by the database so that transactions appear to run sequentially.
4. **Durable** The results of applying a transaction are permanent, even in the presence of failures.