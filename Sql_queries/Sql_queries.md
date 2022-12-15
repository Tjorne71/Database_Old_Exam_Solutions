# Sql Queries

[All Answers](../All_Answers.md)

## August 2022

(a) There are 45 services which have category “Compute”. How many services have category “Analytics”?

```sql
select count(*)
from Service S
	join Category C on S.catid = C.catid
where C.catname = 'Compute';
--45

select count(*)
from Service S
	join Category C on S.catid = C.catid
where C.catname = 'Analytics';
--139
```

(b) How many composite services have no constituent basic service?

```sql
select count (*)
from CompositeService CS
	left join Constitutes C on CS.csid = C.csid
where C.csid is null;
--20
```

(c) The cheapest basic services (with lowest BasicService.usage × Resource.price) are the services with bsid = 35 and bsid = 299. Write a query to return the identifiers of the most expensive basic services.

Note: This query returns a list of (one or more) identifiers.

```sql
drop view if exists BasicServicePrice;
create view BasicServicePrice
as
select  BS.bsid as sid, sum(BS.usage*R.price) as total_price
from BasicService BS
	join Resource R on BS.rid = R.rid
group by BS.bsid;

select BSP.sid
from BasicServicePrice BSP
where BSP.total_price = (
	select min(total_price)
	from BasicServicePrice
);
-- 299, 35

select BSP.sid
from BasicServicePrice BSP
where BSP.total_price = (
	select max(total_price)
	from BasicServicePrice
);
-- 500, 487

drop view if exists BasicServicePrice;
```

(d) The resource with rid = 68 is an example of an over-consumed resource, as its total usage by all basic services is 76, which exceeds its capacity of 62. How many resources are over-consumed in this database?

```sql
drop view if exists ResourceUsage;
create view ResourceUsage
as
select  BS.rid, sum(usage) as total_usage
from BasicService BS
group by BS.rid;

select RU.rid, RU.total_usage, R.capacity
from Resource R
	join ResourceUsage RU on RU.rid = R.rid
where R.rid = 68;
-- 68	76	62

select count(*)
from Resource R
	join ResourceUsage RU on RU.rid = R.rid
where RU.total_usage > R.capacity;
-- 5

drop view if exists ResourceUsage;
```

(e) There are 28 clients which only subscribe to composite services (i.e., they subscribe to some composite services, but no basic services). How many clients only subscribe to basic services?

```sql
select count(distinct SB.cid)
from Subscribes SB
	join CompositeService as CS on SB.sid = CS.csid
where not exists (
	select  *
	from Subscribes as SB2
		join BasicService as BS on SB2.sid = BS.bsid
	where SB.cid = SB2.cid
);
-- 28

select count(distinct SB.cid)
from Subscribes SB
	join BasicService BS on SB.sid = BS.bsid
where not exists (
	select  *
	from Subscribes SB2
		join CompositeService as CS on SB2.sid = CS.csid
	where SB2.cid = SB.cid
);
-- 7
```

(f) There are 2 clients which directly (i.e., not via composite services) subscribe to all basic services offered by the service provider. How many clients subscribe indirectly (i.e., via composite services) to all basic services offered by the service provider?

_Note: This is a division query; points will only be awarded if division is attempted._

```sql
select count(*)
from (
	select S.cid
	from Subscribes S
		join BasicService BS on S.sid = BS.bsid
	group by s.cid
	having count(distinct S.sid) = (
		select count(*)
		from BasicService
	)
) X;
-- 2

select count(*) from (
	select S.cid
	from Subscribes S
		join CompositeService CS on CS.csid = S.sid
		join Constitutes C on C.csid = CS.csid
	group by S.cid
	having count(distinct C.bsid) = (
		select count(*)
		from BasicService
	)
) X;
-- 5
```

(g) The service provider wants to implement the following rule: clients which subscribe to a composite service X should not subscribe to any of the basic services that constitute the composite service X. This rule does not currently hold in the database. As an example, the client with cid = 16 would violate the rule as it subscribes to composite service with csid = 431 and its constituent basic service with bsid = 120 (as well as two other pairs of composite and constituent basic services). How many distinct clients in total would violate this rule?

```sql
select S.cid, CS.csid, BS.bsid
from Subscribes S
	join CompositeService CS on CS.csid = S.sid
	join Subscribes S2 on S.cid = S2.cid
	join BasicService BS on BS.bsid = S2.sid
	join Constitutes C on BS.bsid = C.bsid
where CS.csid = C.csid
	and S.cid = 16;
-- 3 rows, including with 431 / 120

select count(distinct S.cid)
--select distinct S.cid
from Subscribes S
	join CompositeService CS on CS.csid = S.sid
where S.cid in (
	select S2.cid
	from Subscribes S2
		join BasicService BS on BS.bsid = S2.sid
		join Constitutes c on BS.bsid = C.bsid
	-- we cannot have that it is the same client and
	-- that the subscribed basic service constitutes the subscribed composite service
	where S.cid = S2.cid
		and C.csid = CS.csid
);
--order by cid;
-- 79
```

(h) Write a query that returns, for each client that subscribes to some service(s), the cid of the client and the total monthly cost of all its subscriptions.

_Note: This query should return one row, with two columns, for each client that has some subscriptions. You are not asked to paste in the result for this query._

```sql
drop view if exists CompositeServicePrice;
drop view if exists BasicServicePrice;

create view BasicServicePrice
as
select  BS.bsid as sid, sum(BS.usage*R.price) as total_price
from BasicService BS
	join Resource R on BS.rid = R.rid
group by BS.bsid;

create view CompositeServicePrice
as
select  C.csid as sid, SUM(BSP.total_price) as total_price
from BasicServicePrice BSP
	join Constitutes C on C.bsid = BSP.sid
group by C.csid;

-- solution
select S.cid, sum(X.total_price)
from Subscribes as S
join (
	select *
	from BasicServicePrice
	union
	select *
	from CompositeServicePrice
) X on X.sid = S.sid
group by S.cid
order by S.cid;

drop view if exists CompositeServicePrice;
drop view if exists BasicServicePrice;
```

## Maj 2022

(a) There are 461 employees with names that end with “sson” in the database. How many employees have names ending with “sdottir”?

```sql
select count(*)
from employees
where ename like '%sson';
-- 461

select count(*)
from employees
where ename like '%sdottir';
-- 418
```

(b) The number of employees who were at least once audited with a rating above average is 1035. What is the number of the employees who were at least once audited with a rating below the average?

```sql
select count(distinct w.eid)
from audits a
	join works w on a.wid = w.wid
where a.rating > (
	select avg (rating)
	from audits
);
-- 1035

select count(distinct w.eid)
from audits a
	join works w on a.wid = w.wid
where a.rating < (
	select avg (rating)
	from audits
);
-- 933
```

(c) One company policy rule is that projects should be of a type that matches one of the types that clients are registered for. However, client with cid = 1 has 105 projects which violate this rule. How many projects in total violate this rule?

```sql
select count(distinct w.pid)
from works w
	join projects p on p.pid = w.pid
where p.tid not in (
	select ct.tid
	from clients_types ct
	where ct.cid = w.cid
) and w.cid = 1;
-- 105

select count(distinct w.pid)
from works w
	join projects p on p.pid = w.pid
where p.tid not in (
	select ct.tid
	from clients_types ct
	where ct.cid = w.cid
);
-- 151
```

(d) Furthermore, another company policy rule is that employees should not work on different project types for the same client. How many employees violate this rule of the company policy?

```sql
select count(*)
from (
	select distinct w.eid
	from works w
	join projects p on p.pid = w.pid
	group by w.eid, w.cid
	having count(distinct p.tid) > 1
) X;
-- 987
```

(e) The client with cid = 1 has six employees working on the project with pid = 20. Write a query to return the cid attribute of client(s) that have the most number of employees working on a single project?

_Note: This query returns a list of (one or more) identifiers._

```sql
drop view countWorkers if exists;
create view countWorkers
as
select w.cid, w.pid, count (w.eid) as noWorkers
from works w
group by w.cid, w.pid;

select noWorkers
from countWorkers
where cid = 1 and pid = 20;
-- 6

select cid
from countWorkers
where noWorkers = (
	select max(noWorkers) from
	countWorkers
);
-- 2 rows
-- 50
-- 38
drop view countWorkers if exists;
```

(f) Write a query to return the highest total number of hours registered by one worker, across all projects and clients?

_Note: This query returns a sum of hours, not a count of result rows._

```sql
select max(totalHours)
from (
	select sum(w.hours) as totalHours
	from works w
	group by w.eid
) X;
-- 110519
```

(g) In the database, there are two projects where work has been done (worked on) for all clients. For how many different project types has work been done (worked on) for all clients?

_Note: This is a division query; points will only be awarded if division is attempted._

```sql
-- number of project which have been done (worked on) for all clients
select count(*)
from (
	select w.pid
	from works w
	group by w.pid
	having count(distinct w.cid) = (
		select count(*)
		from clients
	)
) X;

-- 2

-- number of project types which have been done (worked on) for all clients
select count(*)
from (
	select p.tid
	from works w
		join projects p on p.pid = w.pid
	group by p.tid
	having count(distinct w.cid) = (
		select count(*)
		from clients
	)
) X;
-- 13
```

(h) In total, there are 7,458 audits where an employee audited some other employee (i.e., not themselves). How many employees have audited all other employees (i.e., excluding themselves)?

_Note: This is also a division query._

```sql
-- number of audits where auditor <> worker
select count(*)
from audits a
	join works w on w.wid = a.wid and w.eid <> a.eid;
-- 7458

select count(*)
from (
	select a.eid
	from audits a
		join works w on w.wid = a.wid and w.eid <> a.eid
	group by a.eid
	having count(distinct w.eid) = (
		select count(*) - 1
		from employees
	)
) X;
-- 1
```

## December 2021

(a) A total of 410 chefs have created at least one recipe. How many have not created any recipes?

```sql
select count(distinct R.created_by)
from recipes R;
-- 410

select count(*)
from chefs C left join recipes R on C.id = R.created_by
where R.created_by is null;
-- 6
```

(b) The chef ‘Foodalicious’ has mastered 56 recipes that have some ingredient(s) of type ‘spice’. How many recipes that have some ingredient(s) of type ‘spice’ has the chef ‘Spicemaster’ mastered?

```sql
select count(distinct M.recipe_id)
from chefs C
	join master M on C.id = M.chef_id
	join use U on M.recipe_id = U.recipe_id
	join ingredients I on U.ingredient_id = I.id
where C.name = 'Foodalicious'
	and I.type = 'spice';
-- 56

select count(distinct M.recipe_id)
from chefs C
	join master M on C.id = M.chef_id
	join use U on M.recipe_id = U.recipe_id
	join ingredients I on U.ingredient_id = I.id
where C.name = 'Spicemaster'
	and I.type = 'spice';
-- 57
```

(c) There are 1,257 recipes in the database with 10 or more steps registered. How many recipes have 3 or fewer steps registered?

```sql
select count(*)
from (
	select S.recipe_id
	from steps S
	group by S.recipe_id
	having count(distinct S.step) >= 10
) X;
-- 1257

select count(*)
from recipes
where id not in (
	select S.recipe_id
	from steps S
	group by S.recipe_id
	having count(distinct S.step) > 3
);
-- 1149
```

(d) How many recipes belong to the same cuisine as at least one of their ingredients?

```sql
select count(distinct R.id)
from recipes R
	join use U on R.id = U.recipe_id
	join belong_to B on B.ingredient_id = U.ingredient_id
where R.belong_to = B.cuisine_id;
```

(e) The recipe with name ‘Fresh Tomato Salsa Restaurant-Style’ has the most steps of all recipes, or 38. What is/are the name of the recipe/s with the most different ingredients of all recipes?

Note: The output of this query is a set of one or more recipe names.

```sql
drop view if exists step_view;
create view step_view
as
select S.recipe_id, count(distinct S.step) as cnt
from steps S
group by S.recipe_id;

select R.name
from step_view S
	join recipes R on S.recipe_id = R.id
where S.cnt = (select max(cnt) from step_view);
-- "Fresh Tomato Salsa Restaurant-Style"

drop view if exists ingr_view;
create view ingr_view
as
select U.recipe_id, count(distinct U.ingredient_id) as cnt
from use U
group by U.recipe_id;

select R.name
from ingr_view S
	join recipes R on S.recipe_id = R.id
where S.cnt = (select max(cnt) from ingr_view);
-- "Dinengdeng"
```

(f) We define the spice ratio of a cuisine as the number of ingredients that belong to it that are of type ‘spice’ divided by the total number of ingredients that belong to the cuisine. Here we consider only cuisines that actually have spices. The highest spice ratio is 1.0, and this spice ratio is shared by 8 cuisines. How many cuisines share the lowest spice ratio?

```sql
drop view if exists spice_count;
create view spice_count
as
select CU.id, count(*) as spice_count
from cuisines CU
	join belong_to B on CU.id = B.cuisine_id
	join ingredients I on B.ingredient_id = I.id
where I.type = 'spice'
group by CU.id;

drop view if exists all_count;
create view all_count
as
select CU.id, count(*) as all_count, SC.spice_count, 1.0*SC.spice_count/count(*) as ratio
from cuisines CU
	join belong_to B on CU.id = B.cuisine_id
	join ingredients I on B.ingredient_id = I.id
	join spice_count SC on CU.id = SC.id
group by CU.id, SC.spice_count;

select count(*)
from all_count
where ratio = (select max(ratio) from all_count);
-- 8

select count(*)
from all_count
where ratio = (select min(ratio) from all_count);
-- 7
```

(g) There are 4,169 recipes that contain some ingredient of all ingredient types. How many recipes contain some ingredient of all ingredient types in the same step?

Note: This is a division query; points will only be awarded if division is attempted.

```sql
select count(*)
from (
	select U.recipe_id --, count(*), count(distinct I.type)
	from use U
		join ingredients I on U.ingredient_id = I.id
	group by U.recipe_id
	having count(distinct I.type) = (
		select count(distinct type)
		from ingredients I
	)
) X;
-- 4169

select count(distinct X.recipe_id)
from (
	select U.recipe_id, U.step, count(*), count(distinct I.type)
	from use U
		join ingredients I on U.ingredient_id = I.id
	group by U.recipe_id, U.step
	having count(distinct I.type) = (
		select count(distinct type)
		from ingredients I
	)
) X;
-- 2722
```

(h) Write a query that outputs the id and name of chefs, and total ingredient quantity (regardless of units), in order of decreasing quantity, for chefs that have created recipes in a cuisine with ‘Indian’ in the name, but only considering ingredients that belong to a cuisine with ‘Thai’ in the name.

```sql
-- (h)

-- My initial solution was missing "distinct",
-- so that error was forgiven
drop view if exists indian_chefs;
create view indian_chefs as
select distinct C.id, C.name
from chefs C
     join recipes R on R.created_by = C.id
     join cuisines CU on CU.id = R.belong_to
where CU.name like '%Indian%';

select C.id, C.name, sum(quantity)
from indian_chefs C
     join recipes R on R.created_by = C.id
     join use U on U.recipe_id = R.id --18m
     join belong_to B on B.ingredient_id = U.ingredient_id
     join cuisines CU on CU.id = B.cuisine_id
where CU.name like '%Thai%'
group by C.id, C.name
order by sum(quantity) desc

-- 328	"Katie Lee"	41
-- 100	"Yutaka Ishinabe"	36
-- 140	"Raymond Oliver"	36
-- 39	"Caesar Cardini, inventor of Caesar salad (1924)"	33
-- 228	"Connie Achurra"	29
-- etc / 150 rows in total
```

## August 2021

(a) There are 8 different plants that are missing the family information. How many plants belong to the family \Thespesia"?

```sql
select count(*)
from plants P
where P.familyID is null;
-- 8

select count(*)
from plants P
	join families F on F.ID = P.familyID
where M.name = 'Thespesia';
-- 18
```

(b) Of the people in the database, 11 have not planted anything. How many of those, who have not planted anything, have the position \Planter"?

```sql
select count(*)
from people E
	left join plantedin I on E.ID = I.planterID
where I.planterID is null;
-- 11

select count(*)
from people E
	left join plantedin I on E.ID = I.planterID
where I.planterID is null
	and E.position = 'Planter';
-- 9
```

(c) The total area of the family \Thespesia" is 66.62 (the unit is square meters; on my machine, the exact number is 66.62000000000003). What is the total area of the family \Vicia"?

_Note: The result needs only be accurate to two digits after the decimal point._

```sql
select sum(1.0 * B.size * I.percentage / 100) as sqm
from families F
	join plants P on F.ID = P.familyID
	join plantedin I on P.ID = I.plantID
	join flowerbeds B on B.ID = I.flowerbedID
where F.name = 'Thespesia';
-- 66.62000000000003
-- 66.62

select sum(1.0 * B.size * I.percentage / 100) as sqm
from families F
	join plants P on F.ID = P.familyID
	join plantedin I on P.ID = I.plantID
	join flowerbeds B on B.ID = I.flowerbedID
where F.name = 'Vicia';
-- 27.3
```

(d) The most over lled owerbed is planted to 105% capacity. What are the ID(s) of the flowerbed(s) with the most over lled capacity?

_Note: The output of this query could contain more than one identi er._

```sql
drop view if exists bedperc;
create view bedperc
as
select I.flowerbedID as ID, sum(I.percentage) as perc
from plantedin I
group by I.flowerbedID;

select max(BP.perc)
from bedperc BP;
-- 105

select BP.ID
from bedperc BP
where BP.perc = (
	select max(BP.perc)
	from bedperc BP
);
-- 243
```

(e) There are 9 owerbeds that are planted to more than 100% capacity. How many flowerbeds are planted to less than 100% capacity.

```sql
select count(*)
from bedperc BP
where BP.perc > 100;
-- 9

select count(*)
from flowerbeds B
where B.ID not in (
	select BP.ID
	from bedperc BP
	where BP.perc >= 100
);
-- 273
```

(f) How many owerbeds are planted to less than 100% capacity, and have a plant of the type \shrub" planted in them?

```sql
select count(*)
from bedperc BP
where BP.perc < 100
 	and BP.ID in (
		select I.flowerbedID
	  	from plantedin I
	  		join plants P on I.plantID = P.ID
			join families F on P.familyID = F.ID
			join types T on F.typeID = T.ID
		where T.name = 'herb'
);
-- 150
```

(g) There are 354 families that are planted in at least one owerbed in all the parks from the database. How many owerbeds have at least one plant of all types from the database.

_Note: This is a division query; points will only be awarded if division is attempted._

```sql
select count(*)
from (
	select F.ID  --,  count(*), count(distinct B.ID), count(distinct B.parkID) -- use these for debugging
	from families F
		join plants P on F.ID = P.familyID
		join plantedin I on P.ID = I.plantID
		join flowerbeds B on B.ID = I.flowerbedID
	group by F.ID
	having count(distinct B.parkID) = (
		select count(*)
		from parks K
	)
) X;
-- 354

select count(*)
from (
	select P.familyID  --,  count(*), count(distinct B.ID), count(distinct B.parkID) -- use these for debugging
	from plants P
		join plantedin I on P.ID = I.plantID
		join flowerbeds B on B.ID = I.flowerbedID
	group by P.familyID
	having count(distinct B.parkID) = (
		select count(*)
		from parks K
	)
) X;
-- 354

select count(*)
from (
	select I.flowerbedID --,  count(*), count(distinct M.ID), count(distinct M.typeID) -- use these for debugging
	from families F
		join plants P on F.ID = P.familyID
		join plantedin I on P.ID = I.plantID
	group by I.flowerbedID
	having count(distinct F.typeID) = (
		select count(*)
		from types T
	)
) X;
-- 2
```

(h) Write a query that returns the ID and name of people, and the total area that they have planted. The list should be restricted to people who have the position \Planter" and who have planted some plant of type \flower" in the park \Kongens Have". The total area returned, however, should not have those restrictions and should represent all the area planted by these people. The list should have the largest area  rst. Note: The readability of the solution is important for this query.

```sql
create view alldata
as
select E.ID, E.name, T.name as type, K.name as park, E.position, 1.0 * B.size * I.percentage / 100 as sqm
from people E
	join plantedin I on E.ID = I.planterID
	join flowerbeds B on B.ID = I.flowerbedID
	join parks K on K.ID = B.parkID
	join plants P on P.ID = I.plantID
	join families F on F.ID = P.familyID
	join types T on T.ID = F.typeID;

select A.ID, A.name, sum(A.sqm)
from alldata A
group by A.ID, A.name
having A.ID in (
	select A2.ID
	from alldata A2
	where A2.type = 'flower'
		and A2.park = 'Kongens Have'
		and A2.position = 'Planter'
)
order by sum(A.sqm) desc;
-- First 5 rows:
-- 154	"Frank Jansen"	72.82
-- 110	"Jan Lauridsen"	72.04999999999998
-- 48	"Johan Mikaelsen"	70.41999999999999
-- 142	"Mikael Lauritz"	67.52999999999999
-- 156	"Mikael Mikaelsen"	67.47
```

## June 2021

(a) The database has one club from the city named Copenhagen. How many clubs are from the city named London?

```sql
select count(*)
from clubs C
	join cities T on C.cityID = T.ID
where T.name = 'Copenhagen';
-- 1

select count(*)
from clubs C
	join cities T on C.cityID = T.ID
where T.name = 'London';
-- 6
```

(b) In the signed with table, there are three different clubID values that no longer exist in the clubs table. How many players signed with those clubs?

```sql
select count(distinct W.clubID)
from signedwith W
where W.clubID not in (
	select C.ID
	from clubs C
);
-- 3

select count(distinct W.playerID)
from signedwith W
where W.clubID not in (
	select C.ID
	from clubs C
);
-- 7
```

(c) The club named Liverpool has a total of 243 away wins, while the highest number of away wins by any club happens to be 260. How many different clubs jointly have the most away wins?

```sql
-- A view to count the away wins per club
drop view if exists awaywins;
create view awaywins
as
select C.ID, C.name, count(*) as awaywins
from matches M
	join clubs C on M.awayID = C.iD
where M.awaywin
group by C.ID;

-- check the view
select *
from awaywins
order by awaywins desc;

select A.awaywins
from awaywins A
where A.name = 'Liverpool';
-- 243

select max(A.awaywins)
from awaywins A;
-- 260

select count(*)
from awaywins A
where A.awaywins = (
	select max(awaywins)
	from awaywins
);
-- 2
```

(d) During the playing career of Andrea Pirlo, he was involved with 319 home goals. As outlined above, this means that while he was signed with di erent clubs, they scored a total of 319 home goals. How many away goals was Steven Gerrard involved with?

```sql
select sum(M.homegoals)
from players P
	join signedwith W on W.playerID = P.ID
	join matches M on W.clubID = M.homeID and W.seasonID = M.seasonID
where P.name = 'Andrea Pirlo';
-- 319

select sum(M.awaygoals)
from players P
	join signedwith W on W.playerID = P.ID
	join matches M on W.clubID = M.awayID and W.seasonID = M.seasonID
where P.name = 'Steven Gerrard';
-- 62
```

(e) During his illustrious playing career, Bjorn signed with 7 different clubs. Write a query to output the name(s) of the player(s) who signed with the largest number of different clubs.

Note: The output of this query is a set of one or more player names.

```sql
drop view if exists clubcount;
create view clubcount
as
select W.playerID, count(distinct W.clubID) as clubs
from signedwith W
group by W.playerID;

select *
from clubcount CC
order by CC.clubs desc;

select CC.clubs
from clubcount CC
	join players P on P.ID = CC.playerID
where P.name = 'Bjorn';
-- 7

select P.name
from clubcount CC
	join players P on P.ID = CC.playerID
where CC.clubs = (
	select max(clubs)
	from clubcount
);
-- Ruud van Nistelrooy
```

(f) How many players never signed with a club from the city named London?

```sql
select count(*)
from (
	select P.ID
	from players P
	except
	select distinct W.playerID
	from signedwith W
		join clubs C on W.clubID = C.ID
		join cities T on C.cityID = T.ID
	where T.name = 'London'
) X;
```

(g) London clubs are de ned here as clubs from the city named London. All 14 non- London clubs have beaten all London clubs away (meaning that the London club was the home team) during some season registered in the database. How many non-London clubs have beaten all London clubs away in a single season?

Note: This is a division query; points will only be awarded if division is attempted.

```sql
select count(*)
from (
	select M.awayID, count(distinct M.homeID)
	from matches M
		join clubs C on M.homeID = C.ID
		join cities T on C.cityID = T.ID
	where T.name = 'London'
		and M.awaywin
	group by M.awayID
	having count(distinct M.homeID) = (
		select count(*)
		from clubs C join cities T on C.cityID = T.ID
		where T.name = 'London'
	)
) X;
-- 14

select count(*)
from (
	select M.awayID, M.seasonID, count(distinct M.homeID)
	from matches M
		join clubs C on M.homeID = C.ID
		join cities T on C.cityID = T.ID
	where T.name = 'London'
		and M.awaywin
	group by M.awayID, M.seasonID
	having count(distinct M.homeID) = (
		select count(*)
		from clubs C join cities T on C.cityID = T.ID
		where T.name = 'London'
	)
) X;
-- 2
```

(h) Given the points rule in the database description, write a query that correctly outputs the  final standings (team names and total points, in descending order of total points) for the season with ID = 2035. The figure below shows the first five lines of the final standings for the season with ID = 2044.

```sql
drop view if exists points;
create view points
as
select M.homeID as clubID, 3 as points, seasonID
from matches M
where M.homegoals > M.awaygoals
union all
select M.homeID, 1, seasonID
from Matches M
where M.homegoals = M.awaygoals
union all
select M.awayID, 1, seasonID
from matches M
where M.homegoals = M.awaygoals
union all
select M.awayID, 3, seasonID
from matches M
where M.homegoals < M.awaygoals;

select name, points
from (
	select C.ID, C.name, sum(S.points) as points
	from points S
		join clubs C on S.clubID = C.ID
	where S.seasonID = 2035
	group by C.ID
	order by sum(S.points) desc
) X;

-- Chelsea	63
-- West Ham	62
-- Fulham	62
-- FCK	61
-- AC	60
-- City	57
-- Juventus	56
-- Liverpool	56
-- Roma	55
-- PSG	54
-- Inter	54
-- Rangers	53
-- Barcelona	53
-- Crystal Palace	48
-- Everton	48
-- Tottenham	45
-- Arsenal	44
-- United	43
-- Bayern	41
-- Real	38
```

## March 2021

(a) The database lists 23 vaccines produced by Amgen. How many vaccines have been produced for the disease Coronavirus?

```sql
-- (a)

select count(*)
from vaccines V
	join companies C on V.comID = C.ID
where C.name = 'Amgen';
-- 23

select count(*)
from vaccines V
	join diseases D on V.disID = D.ID
where D.name = 'Coronavirus';
-- 3
```

(b) There are 282 people that have received some injection of a vaccine for an incurable disease from the category Immune diseases. How many people have received some injection of a vaccine for a curable disease from the category Bone diseases.

```sql
-- (b)

select count(distinct I.peoID)
from injections I
	join vaccines V on V.ID = I.vacID
	join diseases D on D.ID = V.disID
	join categories C on C.ID = D.catID
where D.curable = false
	and C.name = 'Immune diseases';
-- 282

select count(distinct I.peoID)
from injections I
	join vaccines V on V.ID = I.vacID
	join diseases D on D.ID = V.disID
	join categories C on C.ID = D.catID
where D.curable = true
	and C.name = 'Bone diseases';
-- 514
```

(c) The Coronavirus vaccine with the shortest e ect has ID = 535. What is the ID of the Coronavirus vaccine with the longest effect?

_Note: This query returns an identi er, not a count of result rows._

```sql
-- (c)

-- incorrect version: only check disease in inner
select *
from vaccines V
where V.effectyears = (
	select max(V2.effectyears)
	from vaccines V2
		join diseases D on V2.disID = D.ID
	where D.name = 'Coronavirus'
);

-- correct version, make sure the min is only for the coronavirus disease
select V.ID
from vaccines V
	join diseases D on V.disID = D.ID
where D.name = 'Coronavirus'
  and V.effectyears = (select min(V2.effectyears)
					   from vaccines V2
					   where V2.disID = D.ID);
-- 535

select V.ID
from vaccines V
	join diseases D on V.disID = D.ID
where D.name = 'Coronavirus'
  and V.effectyears = (select max(V2.effectyears)
					   from vaccines V2
					   where V2.disID = D.ID);
-- 536
```

(d) How many diseases have fewer than 5 vaccines?

```sql
-- (d)

select count(*)
from diseases D
where D.ID not in (
	select V.disID
	from vaccines V
	group by V.disID
	having count(*) >= 5
);
-- 10
```

(e) According to the database, there are 113 people who have an active vaccination in 2021 for Sleep Apnea. How many people have an active vaccination for Coronavirus in 2021?

_Note: In this query, you are allowed to compare to the number 2021. Alternatively, date part('year', CURRENT DATE) would return 2021 at the time of the exam._

```sql
-- (e)

select count(distinct I.peoID)
from injections I
	join vaccines V on I.vacID = V.ID
	join diseases D on V.disID = D.ID
where D.name = 'Sleep Apnea'
  and I.injectionyear + V.effectyears >= date_part('year', CURRENT_DATE);
-- 113

select count(distinct I.peoID)
from injections I
	join vaccines V on I.vacID = V.ID
	join diseases D on V.disID = D.ID
where D.name = 'Coronavirus'
  and I.injectionyear + V.effectyears >= date_part('year', CURRENT_DATE);
-- 235
```

(f) Person with ID = 23 has had 4 di erent vaccinations. What is the average number of vaccinations for each person in the database?

_Note: This question will accept a range that includes the correct foating point number._

```sql
-- (f)

select count(*)
from People P
	join injections I on P.ID = I.peoID
where P.ID = 23;
-- 4

select (1.0*(select count(*) from injections)
	 /  (select count(*) from people)) as X;
-- 57.4908088235294118

select ((select count(*) from injections)
	 /  (select count(*) from people)) as X;
-- 57
```

(g) There are 150 people who have been vaccinated for all diseases in category Immune diseases. How many people have been vaccinated for at least one disease from each category?

_Note: This is a division query; points will only be awarded if division is attempted._

```sql
-- (g)
select count(*)
from (
	select P.ID, P.name
	from people P
		join injections I on P.ID = I.peoID
		join vaccines V on I.vacID = V.ID
		join diseases D on V.disID = D.ID
		join categories C on D.catID = C.ID
	where C.name = 'Immune diseases'
	group by P.ID
	having count(distinct D.ID) = (
		select count(*)
		from diseases D
			join categories C on D.catID = C.ID
		where C.name = 'Immune diseases'
	)
) X;
-- 150

select count(*)
from (
	select P.ID, P.name
	from people P
		join injections I on P.ID = I.peoID
		join vaccines V on I.vacID = V.ID
		join diseases D on V.disID = D.ID
	group by P.ID
	having count(distinct D.catID) = (
		select count(*)
		from categories C
	)
) X;
-- 450
```

(h) What is the ID of the oldest person which has had injections of the most di erent vaccines for any one curable disease from category Immune diseases?

_Note: This query returns an identi er, not a count of result rows. Having the most di erent vaccines for the same disease is most important; being the oldest is second most important._

```sql
-- (h)

drop view if exists q1h_b;
drop view if exists q1h_a;

create view q1h_a
as
select P.ID as PID, D.ID as DID, P.birthyear, count(distinct V.ID) as vacnum
from people P
	join injections I on P.ID = I.peoID
	join vaccines V on I.vacID = V.ID
	join diseases D on V.disID = D.ID
	join categories C on C.ID = D.catID
where C.name = 'Immune diseases'
	and D.curable = true
group by P.ID, D.ID;

create view q1h_b
as
select Q.PID, Q.birthyear
from q1h_a Q
where Q.vacnum = (select max(Q1.vacnum) from q1h_a Q1);

select Q.PID
from q1h_b Q
where Q.birthyear = (select min(Q1.birthyear) from q1h_b Q1);
-- 422
```

## Januar 2021

(a) The database has 13 models made by the maker VOLVO. How many actual cars have been made by VOLVO?

```sql
-- (a)
-- Number of models made by Volvo
-- => number of cars made by Volvo

select count(*)
from models M
	join makers K on M.makerID = K.ID
where K.name = 'VOLVO';
-- 13

select count(*)
from cars C
	join models M on C.modelID = M.ID
	join makers K on M.makerID = K.ID
where K.name = 'VOLVO';
-- 664
```

(b) Person with ID = 34 has bought cars from 7 different makers. How many different makers has the person with ID = 45 bought from?

```sql
-- (b)
-- Number of different makers of cars that person 34 has bought from
-- => Number of different makers of cars that person 45 has bought from

select count(distinct M.makerID)
from sales S
	join cars C on S.carID = C.ID
	join models M on C.modelID = M.ID
where S.personID = 34;
-- 7

select count(distinct M.makerID)
from sales S
	join cars C on S.carID = C.ID
	join models M on C.modelID = M.ID
where S.personID = 45;
-- 7
```

(c) Six different makers have produced the fewest models, or only one each. What is the ID of the maker which has produced the most models?

_Note: This query returns an identifier, not a count of result rows._

```sql
-- (c)
-- Six different makers with the fewest models (1)
-- => ID of maker with the most models

select makerID
from models M
group by M.makerID
having count(*) = (
	select max(cnt)
	from (
		select count(*) as cnt
		from models M
		group by makerID
	) X
);
-- 10
```

(d) How many cars have been sold fewer than 2 times?

```sql
-- (d)
-- How many cars have been sold < 2 times
-- (some have never been sold, must be included)

-- Find all the cars that were never sold
-- PLUS
-- Find all the cars sold, but fewer times than twice
select count(*)
from (
	select C.ID
	from cars C
		left join sales S on C.ID = S.carID
	where S.carID is null
	union all -- there can be no duplicates...
	select S.carID
	from sales S
	group by S.carID
	having count(*) < 2
) X;
-- 818

select count(*)
from (
	select C.ID
	from cars C
	where C.ID not in (select S.carID from sales S)
	union all -- there can be no duplicates...
	select S.carID
	from sales S
	group by S.carID
	having count(*) < 2
) X;
-- 818

-- Negation: Count all cars, except the ones sold >= 2 times
select count(*)
from (
	select C.ID
	from cars C
	except
	select S.carID
	from sales S
	group by S.carID
	having count(*) >= 2
) X;

-- Negation: Count all cars, except the ones sold >= 2 times
select count(*)
from cars C
where C.ID not in (
	select S.carID
	from sales S
	group by S.carID
	having count(*) >= 2
);
-- 818

-- Doing it all in the select clause
-- NOT A RECOMMENDED SOLUTION :)
select (
	select count(*)
	from cars C
		left join sales S on C.ID = S.carID
	where S.carID is null
) + (
	select count(*)
	from (
		select S.carID
		from sales S
		group by S.carID
		having count(*) < 2
	) X
);
-- 818
```

(e) The car with licence LX363 has been sold eleven times, according to the database. What is the licence of the car sold most often? Note: This query returns a short string.

```sql
-- (e)
-- Car with licence LX363 has been sold twice
-- => What is the licence of the car sold most often

-- helper view that groups by licence and counts sales
drop view if exists licsales;
create view licsales
as
select C.licence, count(*) as salecount
from sales S
	join cars C on S.carID = C.id
group by C.licence;

select LS.salecount
from licsales LS
where LS.licence = 'LX363';
-- 11

select LS.licence
from licsales LS
where LS.salecount = (select max(salecount) from licsales);
-- SI998
```

(f) According to the sellers table, how many people have made at least one sale alone?

```sql
-- (f)
-- How many people have made at least one sale alone

select count(distinct S2.personID)
from (
	select L.saleID
	from sellers L
	group by L.saleID
	having count(*) = 1
) S1 join sellers S2 on S1.saleID = S2.saleID;
-- 544

select count(distinct personID)
from (
	select L.saleID, max(L.personID) as personID
	from sellers L
	group by L.saleID
	having count(*) = 1
) X;
-- 544
```

(g) There are 23 people who have bought cars of all models made by the maker LAM- BORGHINI. How many people have bought cars of all models made by VOLVO?

_Note: This is a division query; points will only be awarded if division is attempted._

```sql
-- (g)
-- Number of people who have bought all models from maker X
-- => Number of people who have bought all models from maker Y

select count(*)
from (
	select S.personID
	from sales S
		join cars C on S.carID = C.ID
		join models M on C.modelID = M.ID
		join makers K on M.makerID = K.ID
	where K.name = 'LAMBORGHINI'
	group by S.personID
	having count(distinct C.modelID) = (
		select count(*)
		from models M
			join makers K on M.makerID = K.ID
		where K.name = 'LAMBORGHINI'
	)
) X;
-- 23

select count(*)
from (
	select S.personID
	from sales S
		join cars C on S.carID = C.ID
		join models M on C.modelID = M.ID
		join makers K on M.makerID = K.ID
	where K.name = 'VOLVO'
	group by S.personID
	having count(distinct C.modelID) = (
		select count(*)
		from models M
			join makers K on M.makerID = K.ID
		where K.name = 'VOLVO'
	)
) X;
-- 5
```

(h) The database is randomly generated, and contains very little quality control relating to the different -year attributes in the various tables. One problem that may arise, for example, is that some cars are produced before the model production started (Cars.prodyear < Models.firstyear ), but there are multiple other problems that might arise. How many different cars have some problem with one of the -year attributes?

_Note: In this query, you must figure out the potential problems and embed them all into a single query. The more (real) problems you find, the higher your score._

```sql
-- (h)
-- How many cars have some problem with years

select count(*)
from (
	select C.ID
	from cars C
		join models M on C.modelID = M.ID
	where M.firstyear > C.prodyear
		or M.lastyear < C.prodyear
	union
	select C.ID
	from cars C
		join sales S on S.carID = C.ID
	where S.saleyear < C.prodyear
	union
	select C.ID
	from cars C
		join sales S on S.carID = C.ID
		join people P on S.personID = P.ID
	where S.saleyear < P.birthyear
	union
	select C.ID
	from cars C
		join sales S on S.carID = C.ID
		join sellers L on L.saleID = S.ID
		join people P on L.personID = P.ID
	where S.saleyear < P.birthyear
) X;
-- 5364
```

## August 2020

(a) There are 286 users which are related to some other users (= have a ‘fromID’). How many users have someone related to them (= have a ‘toID’)?

```sql
-- (a)

-- 286
select count(distinct fromID)
from Relationships;

-- 287
select count(distinct toID)
from Relationships;
```

(b) How many bidirectional relationships exist in the database? Here, if A has a rela- tionship to B and B has a relationship to A that is one bidirectional relationship.

```sql
-- (b)

-- 46
select count(*)
from Relationships A
	join Relationships B on A.toID = B.fromID and A.fromID = B.toID
where A.fromID > B.fromID;
```

(c) In total, there are 1939 entries in the Relationship table. How many of those rela- tionship entries have no associated entry in the Roles table?

```sql
-- (c)

-- 10
select count(*)
from Relationships S
	left join Roles R on R.fromID = S.fromID and R.toID = S.toID
where R.role is null;
```

(d) Needless to say, posts have a varying number of comments (some have none). For example, post 24 has 3 comments. What is the ID of the post that has the most comments? Note: This query returns an identifier, not a count of result rows.

```sql
-- (d)

-- supporting view
drop view if exists postsWithCommentCount;
create view postsWithCommentCount
as
select C.postID as ID, count(*) as cnt
from Comments C
group by C.postID;
```

(e) There are 6 posts which have more than 5 comments. How many posts have 2 or fewer comments?

```sql
-- (e)

-- 6
select count(*)
from postsWithCommentCount C
where C.cnt > 5;

-- 1674
select count(*)
from (
	select C.ID
	from postsWithCommentCount C
	where C.cnt <= 2
	union
	select P.ID
	from Posts P
		left join postsWithCommentCount C on P.ID = C.ID
	where C.cnt is null
) X;
```

(f) In the database, there are 90 different users who have commented on the post of their spouse. How many different municipalities have at least one user who has commented on the post of their spouse?

```sql
-- (f)

-- 90
select count(distinct R.fromID)
from Roles R
	join Comments C on R.toID = C.posterID and R.fromID = C.userID
where R.role = 'Spouse';

-- 33
select count(distinct Z.municipalityID)
from Zips Z
	join Users U on Z.zip = U.zip
	join Roles R on U.ID = R.fromID
	join Comments C on R.toID = C.posterID and R.fromID = C.userID
where R.role = 'Spouse';
```

(g) All in all, you should find that there are 3 different roles in the database. There are 83 users who have a relationship (‘fromID’) to some other users with all the roles in the database. How many users have some other user related to them (‘toID’) with all the roles in the database?

_Note: This is a division query; points will only be awarded if division is attempted. In particular, you must not use the constant 3 in the query._

```sql
-- (g)

-- 83
select count(distinct fromID)
from (
	select R.fromID, R.toID
	from Roles R
	group by R.fromID, R.toID
	having count(role) = (select count(distinct role) from Roles)
) X;

-- 86
select count(distinct toID)
from (
	select R.fromID, R.toID
	from Roles R
	group by R.fromID, R.toID
	having count(role) = (select count(distinct role) from Roles)
) X;
```

(h) How many users have posted a post which has at least one like, but no comments?

_Note: Query complexity will impact the grade strongly for this query_

```sql
-- 206 = readable version
select count(distinct posterID)
from (
	select P.ID, P.posterID
	from Posts P join Likes L on L.postID = P.ID
	except
	select P.ID, P.posterID
	from Posts P join Comments C on P.ID = C.postID
) X;

-- 206 = dense version
select count(distinct posterID)
from Posts P join (
	select L.postID
	from Likes L
	except
	select C.postID
	from Comments C
) X on P.id = X.postID;


-- 206 = simplest student version, probably the most readable!
select count(distinct posterid)
from posts p
where p.id in (select postid from likes)
and p.id not in (select postid from comments)
```

## April 2020

a) There are 22913 employees who were born in 1964. How many employees were born in 1954?

```sql
select count(*)
from employees
where extract(year from birthday) = 1964;
-- 22913

select count(*)
from employees
where extract(year from birthday) = 1954;
-- 23228
```

b) In which year was the current manager of the 'Finance' department born?

```sql
select extract(year from E.birthday)
from employees E
	join dept_manager M on E.emp_no = M.emp_no
	join departments D on M.dept_no = D.dept_no
where D.name = 'Finance'
 	and M.to_date is null;
-- 1957
```

c) There are 3 departments who have had more than 50,000 employees in total. How many departments have more than 50,000 current employees?

```sql
select count(*)
from (
	select dept_no, count(distinct emp_no)
	from dept_emp
	group by dept_no
	having count(distinct emp_no) > 50000
) X;
-- 3

select count(*)
from (
	select dept_no, count(distinct emp_no)
	from dept_emp
	where to_date is null
	group by dept_no
	having count(distinct emp_no) > 50000
) X;
-- 2
```

d) Employee number 429386 is the former employee with the longest time-span in a single department (from '1985-02-03' to '2002-08-01'). What is the employee number of the former employee with the longest total time-span across all departments?

```sql
select emp_no
from dept_emp
where to_date - from_date = (
	select max(to_date - from_date)
	from dept_emp
);
-- 429386

select DE1.emp_no
from dept_emp DE1
	join dept_emp DE2 on DE1.emp_no = DE2.emp_no
where DE2.to_date - DE1.from_date = (
	select max(DE2.to_date - DE1.from_date)
	from dept_emp DE1
		join dept_emp DE2 on DE1.emp_no = DE2.emp_no
);
-- 300023
```

e) In the 'Production' department, there have been in total 8390 employees who have never held any title ending with 'Engineer'. How many such employees have been in total in the 'Development' department?

```sql
select count(*)
from (
	select DE.emp_no
	from dept_emp DE
		join departments D on DE.dept_no = D.dept_no
	where D.name = 'Production'
	except
	select T.emp_no
	from titles T
	where T.title like '%Engineer'
) X;
-- 8390

select count(*)
from (
	select DE.emp_no
	from dept_emp DE
		join departments D on DE.dept_no = D.dept_no
	where D.name = 'Development'
	except
	select T.emp_no
	from titles T
	where T.title like '%Engineer'
) X;
-- 9456
```

f) How many different titles do the current employees of the 'Development' department hold?

```sql
select count(distinct T.title)
from departments D
	join dept_emp DE on D.dept_no = DE.dept_no
	join titles T on DE.emp_no = T.emp_no
where D.name = 'Development'
	and DE.to_date is null;
-- 7
```

g) There are 66263 employees who have held all titles ending with 'Sta '. How many employees have held all titles ending with 'Engineer'?

```sql
select count(*)
from (
	select emp_no, count(*)
	from titles
	where title like '%Staff'
	group by emp_no
	having count(distinct title) = (
		select count(distinct title)
		from titles
		where title like '%Staff'
	)
) X;
-- 66263

select count(*)
from (
	select emp_no, count(*)
	from titles
	where title like '%Engineer'
	group by emp_no
	having count(distinct title) = (
		select count(distinct title)
		from titles
		where title like '%Engineer'
	)
) X;
-- 3008
```

h) Employee number 63966 is the current 'Senior Engineer' in the 'Development' department who has the lowest salary of all such employees. What is the employee number of the current 'Senior Engineer' in the 'Development' department who has the highest salary of all such employees.

```sql
select E.emp_no
from employees E
	join dept_emp DE on E.emp_no = DE.emp_no
	join departments D on DE.dept_no = D.dept_no
	join titles T on T.emp_no = E.emp_no
	join salaries S on S.emp_no = E.emp_no
where DE.to_date is null
	and S.to_date is null
	and T.to_date is null
	and D.name = 'Development'
	and T.title = 'Senior Engineer'
	and S.salary = (
		select min(S1.salary)
		from salaries S1
			join dept_emp DE1 on S1.emp_no = DE1.emp_no
			join departments D1 on DE1.dept_no = D1.dept_no
			join titles T1 on S1.emp_no = T1.emp_no
		where S1.to_date is null
			and DE1.to_date is null
			and D1.name = 'Development'
			and T1.title = 'Senior Engineer'
	);
-- 63966

select E.emp_no
from employees E
	join dept_emp DE on E.emp_no = DE.emp_no
	join departments D on DE.dept_no = D.dept_no
	join titles T on T.emp_no = E.emp_no
	join salaries S on S.emp_no = E.emp_no
where DE.to_date is null
	and S.to_date is null
	and T.to_date is null
	and D.name = 'Development'
	and T.title = 'Senior Engineer'
	and S.salary = (
		select max(S1.salary)
		from salaries S1
			join dept_emp DE1 on S1.emp_no = DE1.emp_no
			join departments D1 on DE1.dept_no = D1.dept_no
			join titles T1 on S1.emp_no = T1.emp_no
		where S1.to_date is null
			and DE1.to_date is null
			and D1.name = 'Development'
			and T1.title = 'Senior Engineer'
	);
-- 86631
```

## Maj 2020

a) There are 15 places with names that end with "borg" in the database. How many places have names ending with "lev"?

```sql
-- 15
select count(*)
from Places
where name like '%borg';

-- 12
select count(*)
from Places
where name like '%lev';
```

b) The ID of the place with the smallest population is 133. What is the ID of the place with the largest population?

```sql
-- 133
select ID
from Places
where population = (
	select min(population)
	from Places
);

-- 34
select ID
from Places
where population = (
	select max(population)
	from Places
);
```

c) Officials now believe that each team should only be registered to one division. However, team with ID of 51, for example, is registered to three divisions. How many teams are registered to more than one division?

```sql
-- 3
select count(*)
from TeamsInDivisions TD
where TD.teamID = 51;

-- 62
select count(*)
from (
	select TD.teamID
	from TeamsInDivisions TD
	group by TD.teamID
	having count(*) > 1
) X;
```

d) Furthermore, officials have realised that teams should only be registered to divisions with the same gender as the team. How many teams are registered to divisions with a di erent gender from the team?

```sql
-- 54
select count(distinct T.ID)
from Teams T
	join TeamsInDivisions TD on T.ID = TD.teamID
	join Divisions D on D.ID = TD.divisionID
where T.genderID <> D.genderID;
```

e) The place with ID of 5 has a total of 15 teams with gender "M" through its various clubs. What is the ID of the place that has the most teams with gender "M"?

```sql
-- supporting view
drop view if exists PlacesWithTeams;
create view PlacesWithTeams
as
select P.ID as PID, P.name,
	   G.ID as GID, G.gender, count(*) as teams
from Places P
	join Clubs C on P.ID = C.placeID
	join Teams T on C.ID = T.clubID
	join Genders G on G.ID = T.genderID
group by P.ID, P.name, G.ID, G.gender;

-- 15
select PT.teams
from PlacesWithTeams PT
where PT.name = 'Aarhus'
  and PT.gender = 'M';

-- 34
select PC.PID
from PlacesWithTeams PC
where PC.teams = (
	select max(teams)
	from PlacesWithTeams
	where gender = 'M'
);
```

f) There are 60 clubs which have teams of all genders. How many places have teams of all genders?

```sql
-- 60
select count(*)
from (
	select T.clubID, count(*)
	from Teams T
	group by T.clubID
	having count(distinct T.genderID) = (
		select count(*)
		from Genders
	)
) X;

-- 30 -- correctly excludes the NULL place
select count(*)
from (
	select P.ID, count(*)
	from Places P
		join Clubs C on P.ID = C.placeID
		join Teams T on C.ID = T.clubID
	group by P.ID
	having count(distinct T.genderID) = (
		select count(*)
		from Genders
	)
) X;

-- 30 -- correctly excludes the NULL place
select count(*)
from (
	select C.placeID, count(*)
	from Clubs C
		join Teams T on C.ID = T.clubID
	where C.placeID is not null
	group by C.placeID
	having count(distinct T.genderID) = (
		select count(*)
		from Genders
	)
) X;

-- 31 -- incorrectly includes the NULL place
select count(*)
from (
	select C.placeID, count(*)
	from Clubs C
		join Teams T on C.ID = T.clubID
	-- where C.placeID is not null
	group by C.placeID
	having count(distinct T.genderID) = (
		select count(*)
		from Genders
	)
) X;
```

g) How many clubs have teams registered to all divisions of the "M" gender?

```sql
-- 1
select count(*)
from (
	select T.clubID, count(*)
	from Teams T
		join TeamsInDivisions TD on T.ID = TD.teamID
		join Divisions D on D.ID = TD.divisionID
		join Genders G on G.ID = D.genderID
	where G.gender = 'M'
	group by T.ClubID
	having count(distinct TD.divisionID) = (
		select count(*)
		from Divisions D
			join Genders G on G.ID = D.genderID
		where G.gender = 'M'
	)
) X;
```

h) Teams earn points in sets. For example, the team with ID of 0 has earned a total of 268 points. (Of those, 167 are home points, while 101 are away points; this division is not important, except to remind you to consider both home points and away points). What is the highest number of points earned by any one team?

```sql
-- supporting view
drop view if exists TeamsWithPoints;
create view TeamsWithPoints
as
select ID, sum(points) as points
from (
	select T.ID, sum(S.homepoints) as points
	from Teams T
		join Matches M on T.ID = hometeamID
		join Sets S on M.ID = S.matchID
	group by T.ID
	union all
	select T.ID, sum(S.awaypoints) as points
	from Teams T
		join Matches M on T.ID = awayteamID
		join Sets S on M.ID = S.matchID
	group by T.ID
) X
group by ID;

-- 268
select *
from TeamsWithPoints
where ID = 0;

-- 1637
select max(points)
from TeamsWithPoints;
```
