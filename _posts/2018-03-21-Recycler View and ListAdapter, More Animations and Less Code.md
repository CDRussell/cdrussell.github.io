---
layout: post
title: "RecyclerView — More Animations with Less Code using Support Library ListAdapter"
description: ListAdapter is a new class bundled in the 27.1.0 support library and simplifies the code required to work with RecyclerViews

date: 2018-03-21

# featured image will be shown in the hero pane, and on the blog post summary views
#featured_image: /images/image.jpg

# images listed appear will be used when sharing to twitter
images:
#- /images/image.jpg

categories:
  - Android

permalink: 2018/03/recyclerview-more-animations-with-less-code-using-support-library-listadapter/
---

*This post describes how to use `ListAdapter`, a new addition from the Support Library, which simplifies the code required to work with `RecyclerViews`.*

[`ListAdapter`](https://developer.android.com/reference/android/support/v7/recyclerview/extensions/ListAdapter.html?source=post_page---------------------------) is a new class bundled in the [27.1.0 support library](https://developer.android.com/topic/libraries/support-library/revisions.html?source=post_page---------------------------#27-1-0) and simplifies the code required to work with RecyclerViews. The end result is less code for you to write and more `RecyclerView` animations happening for free.

Why is it great? It’s great because it automatically stores the previous list of items and utilises [DiffUtil](https://developer.android.com/reference/android/support/v7/util/DiffUtil.html?source=post_page---------------------------) under the hood to only update items in the recycler view which have changed. This typically means better performance as you avoid refreshing the whole list, and nicer animations because only items which change need to be redrawn.

🎉 Since the new [ListAdapter](https://developer.android.com/reference/android/support/v7/recyclerview/extensions/ListAdapter.html?source=post_page---------------------------) is built on top of `DiffUtil` it leverages it for you, meaning you have less callbacks to implement to achieve recycler view animations.

Let’s say you have this class which represents a user’s tasks in a todo app. This example happens to be a [Room entity](https://developer.android.com/topic/libraries/architecture/room.html?source=post_page---------------------------) but that’s not required.

```kotlin
@Entity
data class Task(@PrimaryKey(autoGenerate = true) 
                var id: Int = 0, 
                var title: String = "", 
                var completed: Boolean = false)
```

## Adapter — The Old Way

To display a list of tasks in a `RecyclerView`, previously you would have written your adapter to inherit from `RecyclerView.Adapter<ViewHolder>` and it might have looked something like this.

```kotlin
class TaskListAdapter : RecyclerView.Adapter<ViewHolder>() {

    private val tasks: MutableList<Task> = ArrayList()

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
        val inflater = LayoutInflater.from(parent.context)
        return ViewHolder(inflater.inflate(R.layout.item_task_row, parent, false))
    }

    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        holder.bind(tasks[position])
    }

    override fun getItemCount(): Int = tasks.size

    fun addTask(task: Task) {
        if (!tasks.contains(task)) {
            tasks.add(task)
            notifyItemInserted(tasks.size)
        }
    }

    class ViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {

        fun bind(task: Task) {
            itemView.taskTitle.text = task.title
        }

    }
}
```

The biggest problem this code has is that it only handles adding new tasks and wouldn’t cope with deletions or edits. If we wanted to easily handle those cases as well, we could cheat and use [notifyDataSetChanged()](https://developer.android.com/reference/android/support/v7/widget/RecyclerView.Adapter.html?source=post_page---------------------------#notifyDataSetChanged()) but we’d lose the nice animations. To handle them properly, we’d need to implement `DiffUtil` which is a bit more work.

## DiffUtil
If you use `ListAdapter` you won’t have to fully implement `DiffUtil`, but it’s still useful to know what problem is being solved for you.

`ListAdapter` is built on top of `DiffUtil` meaning it handles some of the diffing logic for you. If you aren’t familiar with `DiffUtil`, it was created to further encourage developers to let the `RecyclerView` only update the contents which changed; resulting in better performance when items are inserted, updated or deleted.

If you are still using `notifyDataSetChanged()` then you are forcing the entire list to be redrawn and therefore not allowing Android the chance to show nice visual cues to the user as to what exactly just changed. Instead of reloading the entire list, you can use methods like `notifyItemInserted`, `notifyItemChanged` and `notifyItemRemoved`. These result in lovely animations but working out which method to invoke manually is non-trivial.

`DiffUtil` helps you by calculating which methods to invoke out of inserted/changed/removed.

`DiffUtil` is great but it still requires a bit of boilerplate to implement.

## Adapter — The New Way
To make use of the new `ListAdapter` we would update the existing adapter so that it inherits `ListAdapter` and pass in the type of item it will show — Task in this case — as well as the `ViewHolder`.

We no longer need to hold onto a reference of our item list as `ListAdapter` does this for you. Whenever you need your item list, you can call `getItem(int)`.

### DiffUtil.ItemCallback
In the constructor, you need to provide a [`DiffUtil.ItemCallback`](https://developer.android.com/reference/android/support/v7/util/DiffUtil.ItemCallback.html?source=post_page---------------------------) which is a slimmed down version of the `DiffUtil` callbacks.

```kotlin
class TaskDiffCallback : DiffUtil.ItemCallback<Task>() {
    override fun areItemsTheSame(oldItem: Task?, newItem: Task?): Boolean {
        return oldItem?.id == newItem?.id
    }

    override fun areContentsTheSame(oldItem: Task?, newItem: Task?): Boolean {
        return oldItem == newItem
    }
}
```

`DiffUtil.ItemCallback` requires you to implement two functions: `areItemsTheSame` and `areContentsTheSame`

### areItemsTheSame
`areItemsTheSame` hands you two items and asks you if they fundamentally represent the same object. In my case, I have a unique ID I can use to compare. If the IDs match, then I know the items are the same even if some of their other fields may differ.

### areContentsTheSame
`areContentsTheSame` hands you two items again. This time, though, it asks you if the two items have the same content. If you decide your items do have the same content, then no redraw and no animation are required; you are already displaying the object and you are saying that nothing has changed.

However, if you decide the contents are not the same, then the item will be redrawn on screen.

## Shiny New Adapter
It’s not much different from the old adapter. We have a new superclass, we’ve deleted the list of items we were storing previously and we’ve ditched the manual `addTask()` method we had before. In the constructor, we pass in an instance of our `DiffUtil.ItemCallback`.

```kotlin
class TaskListAdapter(private val clickListener: (Task) -> Unit) :
    ListAdapter<Task, ViewHolder>(TaskDiffCallback()) {

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
        val inflater = LayoutInflater.from(parent.context)
        return ViewHolder(inflater.inflate(R.layout.item_task_row, parent, false))
    }

    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        holder.bind(getItem(position), clickListener)
    }
        
    class ViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {

        fun bind(task: Task, clickListener: (Task) -> Unit) {
            itemView.taskTitle.text = task.title
            itemView.setOnClickListener { clickListener(task) }
        }
    }
}
```

## Updating the Adapter
Whenever you need to update the adapter, you can provide the new list of items to it using the `submitList(List)` method.

You aren’t tied into using `Room` or `LiveData` and so long as you can obtain an updated list, you can make use of `ListAdapter`.

```kotlin
taskDao.getAll().observe(this, Observer<List<Task>> {
    taskListAdapter.submitList(it)
})
```

But it sure is nice when you do use `Room` and `LiveData` 😎. Whenever you are notified of a change, just submit that straight to the adapter and it’ll take care of the rest.

## Summary
`RecyclerView` is great. But it’s better when animations are shown as its contents are changed. You can cheat and update the entire list every time something changes but that’s usually not necessary. You can achieve better performance and those nice animations yourself with a bit of code.

Or you can make use of [ListAdapter](https://developer.android.com/reference/android/support/v7/recyclerview/extensions/ListAdapter.html?source=post_page---------------------------). The end result is less code, more animations and happier users 😃.