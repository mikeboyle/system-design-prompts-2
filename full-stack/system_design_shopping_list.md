# System Design Intervew: Shopping list app
You've been asked to design a shopping list application.

People will use this app to create a shopping list, for example for the supermarket.

Start with the first set of user stories and describe how we'd implement this. Then move on to the other sets of user stories.

It is normal to revise your ideas as you encounter new user stories.

Finish all of the user stories is not the primary objective. Take your time making your thoughts clear and explaining your choices.

## User stories 1 - basic list
1. I can create a shopping list.
1. I can give my list a name.
1. I can add items to the shopping list.
1. I can mark when an item on the list has been purchased.

## User stories 2 - item management
1. I can edit and delete an item on the list.
1. I can set the desired **quantity** of an item. (Ex: 3 apples, 2 bottles of soda)
1. I can set the desired **amount** and **units** of an item (Ex: 2 pounds of potatoes, 300 grams of cheese)

## User stories 3 - list management
1. I can create many different lists (not just one list).
1. I can delete a list.
1. I can create a new list by "cloning" (copying) another list.

## User stories 4 - collaboration
1. I can add another user as a collaborator to a list I create. (Assume that the user has access to the ids of other registered users.)
1. A collaborator can add, edit, and delete items from the shared list.
1. A collaborator can change the name of the list.
1. A collaborator can add other collaborators.
1. A collaborator **cannot** delete the list (only the list creator can do that).
1. I can see the lists I created as well as the lists I collaborate on. (And I can tell which is which.)