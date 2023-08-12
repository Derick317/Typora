# Pintos Debug

## List Remove All Elements

If you want to get all elements in a `list`, perhaps process them and eventually remove them, you should **not** call `list_remove` before `elem = list_next(elem)`. 

The following code is **WRONG**, since after removing the element from the list, `list_next(elem)` is invalid.

```c
while (elem != list_tail(children))
{
    child_thread = list_entry(elem, struct thread, child_elem);
    child_thread -> parent = NULL;
    list_remove (elem);
    if (child_thread->status == THREAD_DYING)
    {
        ASSERT (list_empty(&child_thread->children_list));
        palloc_free_page (child_thread);
    }
    elem = list_next(elem);
}
```

Unfortunately, the code below is sill **WRONG** (note `list_remove` removes ELEM from its list and returns the element that followed it), because when `child_thread` is freed, `elem` cannot be access again!

```c
while (elem != list_tail(children))
{
    child_thread = list_entry(elem, struct thread, child_elem);
    child_thread -> parent = NULL;
    if (child_thread->status == THREAD_DYING)
    {
        ASSERT (list_empty(&child_thread->children_list));
        palloc_free_page (child_thread);
    }
    elem = list_remove(elem);
}
```

The correct code is

```c
while (elem != list_tail(children))
{
    child_thread = list_entry(elem, struct thread, child_elem);
    child_thread -> parent = NULL;
    elem = list_remove(elem);
    if (child_thread->status == THREAD_DYING)
    {
        ASSERT (list_empty(&child_thread->children_list));
        palloc_free_page (child_thread);
    }
}
```

