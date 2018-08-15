# Zapier_MAU_SkillsTest
Zapier skills test

Datagrip was the main platform used in the creation of the data model.

To start creating the data model, I imported a dates table named dimdates that identified several date attributes like Quarters, Months, and Years to gain additional flexibility while working with dates and to use the unique dates as a forgein key to other tables. The import was completed using the GUI Import date from file option from a csv I previously created.


The original files in the Source_date were copied to the amahaley schema for read and write capibilites. The task_used_da table was altered in two ways. First, the column header date was changed to activity_date to prevent using a SQL reserved word.The column name change was completed using the GUI under the modify table option. The Second, the table was altered to included a foreign key to the dimdates table.The same process was taken for the task_used table, the column header was changed to date_activity and the table was altered to include a foreign key. See code below.

   ALTER TABLE tasks_used_da
   ADD CONSTRAINT tasks_used___fk
   FOREIGN KEY (activity_date) REFERENCES dimdates (date) ;
   
   ALTER TABLE tasks_used
   ADD CONSTRAINT tasks_used___fk
   FOREIGN KEY (date_activity) REFERENCES dimdates (date) ;

Information is usually easier to grasp in a summary format. 
After the tables were linked by Primary Key to Foreign Key. A summary was created based on the dates, daily active users, and the Sum of task for each activity_date. See code below.

   SELECT activity_date , count(distinct  user_id) as dau , sum(sum_tasks_used) as Sum_of_tasks_used
   from amahaley.tasks_used_da
   group by 1
   order by 1;
   
   
Additional Queries used to better understand the date

Used to create a Master View of the data. Hide column to clean presentation

  SELECT * from tasks_used tu inner join tasks_used_da tud on
      tu."user_id " = tud.user_id
  limit 500;

Used to create a list of user who hade no activity for a day

  Select distinct (user_id),Count(user_id) from tasks_used_da
  Where sum_tasks_used = 0
  GROUP BY user_id

Used to create a previous 28 day forecast for active users.

  With dau as ( select date, count(distinct user_id) as dau
      From tasks_used_da tud
      GROUP BY date)

  select date , dau,
      (SELECT count(distinct user_id)
                      from tasks_used_da tud
                        where tud.date between date - 29 * interval '28 day' and date ) as mau
  from dau
