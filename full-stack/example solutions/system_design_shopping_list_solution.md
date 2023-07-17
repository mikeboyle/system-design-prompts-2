# System Design Intervew: Shopping list app
You've been asked to design a shopping list application.

People will use this app to create shopping lists, for example for the supermarket.

Start with the first set of user stories and describe how we'd implement this. Then move on to the other sets of user stories.

It is normal to revise your ideas as you encounter new user stories.

Finishing all of the user stories is not the goal. Take your time making your thoughts clear and explaining your choices.

## User stories 1 - basic list
1. I can create many shopping lists.
1. I can give each shopping list a name.
1. I can add items to a shopping list.
1. I can mark when an item on a list has been purchased.
1. I can edit a shopping list's name.
1. I can delete a shopping list.

## User stories 2 - item management
1. I can edit and delete an item on the list.
1. I can set the desired **quantity** of an item. (Ex: 3 apples, 2 bottles of soda)
1. I can set the desired **amount** and **units** of an item (Ex: 2 pounds of potatoes, 300 grams of cheese)

## User stories 3 - collaboration
1. I can add another user as a collaborator to a list I create. (Assume that the user has access to the ids of other registered users.)
1. A collaborator can add, edit, and delete items from the shared list.
1. A collaborator can change the name of the list.
1. A collaborator can add other collaborators.
1. A collaborator **cannot** delete the list (only the list creator can do that).
1. I can see the lists I created as well as the lists I collaborate on. (And I can tell which is which.)

## Example solution
### Schema
The main entities are `users`, `lists`, and `items`.

- A `list` has one `owner` user, and a user can own many lists.
- A `list` has many `items`.
- An item has a quantity and a unit (which can be null if we just mean x number of the thing).
- `collaborations` reference the list and the users who granted and received the right to collaborate.

![Example schema diagram](./Shopping%20List%20Example%20Solution.svg)

### SQL
Candidates won't be expected to write SQL but should describe generally how the schema would support the user stories.

Examples:
- See all the lists I own: `SELECT * FROM lists WHERE owner_id = $1`
- See all the lists where I can collaborate:
    ```
    SELECT lists.*
    FROM collaborations
    JOIN lists
    ON collaborations.list_id = lists.id
    WHERE invitee_id = $1;
    ```
- Since we're only interested in retrieving data about the list could maybe also be done with a subquery:
    ```
    SELECT *
    FROM lists
    WHERE id
    IN (
        SELECT list_id
        FROM collaborations
        WHERE invitee_id = $1
    );
    ```

### Endpoints
|Endpoint | Methods | Notes|
|---------| ------- | -----------|
|`/users`| `POST`, `GET` | Create a user, show a list of users | 
|`/users/:id`|`GET`, `PUT`, `DELETE`| Display and edit a user; do not allow users to edit other users |
|`/users/:id/lists`| `GET` | Show the lists a user owns (or collaborates on); allow for query string filtering |
|`/lists`|`POST`| Create a list |
|`/lists/:id`|`GET`, `PUT`, `DELETE`|User can only edit/delete lists they own; this route is for editing list name, etc. but not for adding / removing / editing items|
|`/lists/:id/items`|`GET`|Show the items for a list|
|`/items`|`POST`|Add an item to a list|
|`/items/:id`|`PUT`, `DELETE`|Mark an item as (un)purchased, or edit an item; only owners and collaborators can do this|


### Roles and permissions
Many of the operations listed above are restricted to users who own a list or are collaborators on a list.

We can implement these checks in the application, for example with middleware functions.

- `checkOwnerMiddleware`: for example for `DELETE /lists/:id` (only owners allowed)
    - take the list `id` from the route params
    - get the requesting user's `id` (assume we have an authentication system in place that provides this with the request, ex: with a cookie)
    - query the list's `owner_id` in the database
    - if the `owner_id` does not match the user id, return a 403.
    - if the user id can't be found, return a 401.

- `checkOwnerOrCollaboratorMiddleware`: similar to above but would be invoked on routes that allow an owner or collaborator, ex: `PUT /items/:id` or `PUT /lists/:id`
    - if the list id is not in the route params, query the item to find the list_id
    - query the list to get the owner_id
    - if the user id equals owner_id, skip the rest of this 
    - else: query the collaborations where list_id === this list id and invitee_id === the current user id
    - if a collaboration cannot be found, return a 403


