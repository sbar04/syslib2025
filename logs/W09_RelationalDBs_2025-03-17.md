# Week 9: Relational Databases
Relational databases manage and retrieve data efficiently by spreading data across multiple tables. Breaking data into multiple tables can reduce redundancy and improve consistency.

Using __foreign keys__, different tables in the same database can be linked together for better data retrieval. 

## Creating a Database
This is mostly repeat of last week. 

Login in as the root user, who has the privilege to create new databases:
```
sudo mysql -u root
```
Create a new database:
```
create database DinnerDB;
```
Grant privileges to `opacuser` account:
```
grant all privileges on DinnerDB.* to 'opacuser'@localhost';
```
And exit:
```
mysql> \q
```
## Creating Tables
Login as `opacuser`:
```
mysql -u opacuser -p
```
>*See password in the `/var/www/login.php` file created last week.*

Make sure the new `DinnerDB` database is available:
```
[(none)]> show databases;
```
And switch into `DinnerDB` database:
```
[(none)]> use DinnerDB;
```
### Create Meals Table
Use `create` command to create the table:
```
create table Meals (
    meal_id int auto_increment primary key,
    meal_name varchar(100) not null,
    cuisine varchar(50),
    cooking_time int not null default 1 check (cooking_time > 0),
    vegetarian boolean
);
```
>*Similar syntax to last week's tables. New this week is the `check` constraint for `cooking-time` which will reject a 0 or negative value. It also defaults to 1 if no value is entered.*

>*The BOOLEAN value for `vegetarian` must be TRUE or FALSE.*

### Create Ingredients Table
Same as above, but with ingredients:
```
create table Ingredients (
    ingredient_id int auto_increment primary key,
    meal_id int,
    ingredient_name varchar(100) not null,
    quantity varchar(50),
    foreign key (meal_id) references Meals(meal_id) on delete cascade
);
```
>*The __`foreign key`__ allows us to reference the `meal_id` __primary key__ in the `Meals` table from this same database. This makes the `Ingredients` table a child of the `Meals` table.*

>*The `on delete cascade` clause makes it so that if a meal in the `Meals` table is deleted, so are the associated ingredients in the `Ingredients` table.*

## Inserting Data
Added the following to the `Meals` table:
```
insert into Meals (meal_name, cuisine, cooking_time, vegetarian) values
    ('Spaghetti Bolognese', 'Italian', 45, FALSE),
    ('Vegetable Stir Fry', 'Chinese', 20, TRUE),
    ('Chicken Curry', 'Indian', 50, FALSE),
    ('Mushroom Risotto', 'Italian', 35, TRUE);
```
Use `select * from Meals;` to see the `meal_id` for each meal.

Added the following to the `Ingredients` table, using the `meal_id` from the `Meals` table in the first column:
```
insert into Ingredients (meal_id, ingredient_name, quantity) values
    (1, 'Spaghetti', '200g'),
    (1, 'Ground Beef', '250g'),
    (1, 'Tomato Sauce', '1 cup'),
    (2, 'Broccoli', '100g'),
    (2, 'Carrots', '50g'),
    (2, 'Soy Sauce', '2T'),
    (3, 'Chicken Breast', '300g'),
    (3, 'Curry Powder', '2T'),
    (3, 'Coconut Milk', '1 cup'),
    (4, 'Arborio Rice', '1 cup'),
    (4, 'Mushrooms', '1 cup'),
    (4, 'Parmesan Cheese', '1/2 cup');
```
## Querying Data
Practiced using different query commands:

`Select` can be very basic:
```
select * from Meals;
```
But adding in other elements can return cleaner and more complex data. 

Here, the results are filtered by the `where` clause.
```
select * from Meals where vegetarian = TRUE;
```
Here, the results are sorted by the `order by` clause:
```
select * from Meals order by cooking_time desc; 
```

Here, I use both `as` and `join` to rename columns and join information from the two tables in the `DinnerDB` database:
```
select Meals.meal_name as Meals,
    Ingredients.ingredient_name as Ingredients,
    Ingredients.quantity as Quantity
    from Meals
    join Ingredients on Meals.meal_id = Ingredients.meal_id;
```
>*Using `as` renames the column indicated.*

>*The `join` action allows cross-referencing between tables based on the shared `meal_id` value.*

I can search just for the ingredients of a meal with `where`:
```
select ingredient_name as Ingredients,
    quantity as Quantity
    from Ingredients 
    where meal_id = (select meal_id from Meals where meal_name = 'Chicken Curry');
```
Or provide the count of Meals by type of cuisine with `count`:
```
select cuisine, count(*) as meal_count 
    from Meals
    group by cuisine;
```

Or see meals with a cook time equal to or under 45 minutes with `where`, and sort them by ascending order with `order by`:
```
select meal_name, cooking_time 
    from Meals 
    where cooking_time <= 45
    order by cooking_time asc;
```

## Database Management
### Revoking Privileges
Login as root MySQL user:
```
sudo mysql -u root
```
Show privileges for `opacuser`:
```
mysql> show grants for 'opacuser'@'localhost';
```
Take away privileges with `revoke` command:
```
mysql> revoke all privileges on DinnerDB.* from 'opacuser'@'localhost';
```
And confirm with `show grants` command.

### Tracking Users
Use the `select` command to pull from the `user` table in the `mysql` database:
```
select user, host from mysql.user;
```

### Deleting Databases
Use the `drop` command:
```
mysql> drop database DinnerDB;
```
### Deleting Users
Use the `drop` command:
```
mysql> drop user 'sean'@'localhost';
```

## Takeaways
This was very fun! I want to add in more tables to better understand how this all works, and keep trying new queries. 

I played around making a few different databases, but I still need a lot of practice with MySQL syntax. 
