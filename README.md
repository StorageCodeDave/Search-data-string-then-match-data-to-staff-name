## Finds who added or deleted a holiday out of the database

I had a request that holidays booked in the system where been deleted. Many Users are using the interactive GUI and had no idea who was deleting them as a delete command had removed the holiday completely. There is an Audit table which can be read but it has an audit for everything and only shows the Empref (UniqueID) which then has to be match to the first and last name to be useful.
Table to search in
![image](https://github.com/user-attachments/assets/526abbe8-d381-488b-8ae8-58809ff3eb6d)

 
I wrote this script to return the section of string with the empref in and then match it so I could see who the person was. I also allowed the search variable to be the surname or if ALL was inputted it returned everyone. The match had to be on the Empref and data that returned the name in the string and no Empref.

To find the deleted then I had to look for the person’s name been selected and then on the next row if delete was pressed.
![image](https://github.com/user-attachments/assets/83427a60-405f-4dea-ae27-e00714375a88)
 

I used charindex to populate the string and union all to gather various searches into 1 format. I wanted to match on name and then lag 9 rows looking for ‘Delete’ in the string as User might of paused before deleting the person and the audit filled up with other data. I can then easily read the Audit table and check the ChangedBy column to who had added or removed any Holidays



Declare @Surname Varchar (50)
/* Use Surname to locate staff or use 'ALL' to select all staff
	Note when you view all staff the order is by ChangedBy 
	so you can what each Realtime user did and what order of
	actions they did*/


Set @Surname = 'ALL'
;



With CTE as
(
	Select audtime,changedby,audtxt,right(audtxt, len(audtxt) - 43) As Empref,NULL as FirstName,NULL as Surname1 from auddetails 
	where audtxt like 'Book Absence - AbsDate%' 
			or audtxt like 'DeleteBaseListRec: SQL=Delete from Absences%' 
			or audtxt like 'UpdateBaseListRec: SQL=Update Entitlements%'
UNION ALL
		Select audtime,changedby,audtxt,NULL,NULL,NULL

	from auddetails 
	where 
		audtxt like 'Delete Button%' 

	
Union ALL
	Select audtime,changedby,audtxt,NULL

		,SUBSTRING(audtxt,
						CHARINDEX(',',(audtxt))+ 1
													,CHARINDEX(',', audtxt)
																			)
																										as FirsName 
		,SUBSTRING(audtxt, -- 1 -Use this column
						CHARINDEX(' ',(audtxt))+ 1  -- 2 -Find 1st space and start
													,CHARINDEX(',', audtxt)  -- 3 - Length is number but we want to measure where comma is
																			-CHARINDEX(' ',audtxt)-1) -- 4 - Optional but need to deduct where 1st space is to avoid length beentoo long
																										as Surname 
		from auddetails 
			where 
				audtxt like 'select%' 
) 

-- Now Select and join to the Empref to find First and Last Name

Select 
CONVERT(VARCHAR(8), audtime, 3) AS [Date],CONVERT(VARCHAR(5), audtime, 108) AS [Time],
CTE.changedby,CTE.audtxt,coalesce(CTE.Empref,E.empref,00) as Empref--,coalesce (E.forenames,CTE.firstname,'DELETED') As FirstName,coalesce(E.surname,CTE.surname1,'')as Surname
,
case 
	When coalesce (E.forenames,CTE.firstname) > '' Then coalesce (E.forenames,CTE.firstname)

	when coalesce (E.forenames,CTE.firstname,'DELETED') = 'DELETED' and Lag(coalesce (E.forenames,CTE.firstname),1,0) over (order by CTE.audtime) IS NOT NULL then
	coalesce(Lag(coalesce (E.forenames,CTE.firstname,'DELETED'),1,0) over (order by CTE.audtime),'ACTIVE1')

	when coalesce (E.forenames,CTE.firstname,'DELETED') = 'DELETED' and Lag(coalesce (E.forenames,CTE.firstname),2,0) over (order by CTE.audtime) IS NOT NULL then
	coalesce(Lag(coalesce (E.forenames,CTE.firstname,'DELETED'),2,0) over (order by CTE.audtime),'ACTIVE1')

	when coalesce (E.forenames,CTE.firstname,'DELETED') = 'DELETED' and Lag(coalesce (E.forenames,CTE.firstname),3,0) over (order by CTE.audtime) IS NOT NULL then
	coalesce(Lag(coalesce (E.forenames,CTE.firstname,'DELETED'),3,0) over (order by CTE.audtime),'ACTIVE1')

	when coalesce (E.forenames,CTE.firstname,'DELETED') = 'DELETED' and Lag(coalesce (E.forenames,CTE.firstname),4,0) over (order by CTE.audtime) IS NOT NULL then
	coalesce(Lag(coalesce (E.forenames,CTE.firstname,'DELETED'),4,0) over (order by CTE.audtime),'ACTIVE1')

	when coalesce (E.forenames,CTE.firstname,'DELETED') = 'DELETED' and Lag(coalesce (E.forenames,CTE.firstname),5,0) over (order by CTE.audtime) IS NOT NULL then
	coalesce(Lag(coalesce (E.forenames,CTE.firstname,'DELETED'),5,0) over (order by CTE.audtime),'ACTIVE1')

	when coalesce (E.forenames,CTE.firstname,'DELETED') = 'DELETED' and Lag(coalesce (E.forenames,CTE.firstname),6,0) over (order by CTE.audtime) IS NOT NULL then
	coalesce(Lag(coalesce (E.forenames,CTE.firstname,'DELETED'),6,0) over (order by CTE.audtime),'ACTIVE1')

	when coalesce (E.forenames,CTE.firstname,'DELETED') = 'DELETED' and Lag(coalesce (E.forenames,CTE.firstname),7,0) over (order by CTE.audtime) IS NOT NULL then
	coalesce(Lag(coalesce (E.forenames,CTE.firstname,'DELETED'),7,0) over (order by CTE.audtime),'ACTIVE1')

	when coalesce (E.forenames,CTE.firstname,'DELETED') = 'DELETED' and Lag(coalesce (E.forenames,CTE.firstname),8,0) over (order by CTE.audtime) IS NOT NULL then
	coalesce(Lag(coalesce (E.forenames,CTE.firstname,'DELETED'),8,0) over (order by CTE.audtime),'ACTIVE1')

	when coalesce (E.forenames,CTE.firstname,'DELETED') = 'DELETED' and Lag(coalesce (E.forenames,CTE.firstname),9,0) over (order by CTE.audtime) IS NOT NULL then
	coalesce(Lag(coalesce (E.forenames,CTE.firstname,'DELETED'),9,0) over (order by CTE.audtime),'ACTIVE1')																
																	else '' END As StaffName1
,
case 
	When coalesce (E.surname,CTE.surname1) > '' Then coalesce (E.surname,CTE.surname1)

	when coalesce (E.surname,CTE.surname1,'DELETED') = 'DELETED' and Lag(coalesce (E.surname,CTE.surname1),1,0) over (order by CTE.audtime) IS NOT NULL then
	coalesce(Lag(coalesce (E.surname,CTE.surname1,'DELETED'),1,0) over (order by CTE.audtime),'ACTIVE1')

	when coalesce (E.surname,CTE.surname1,'DELETED') = 'DELETED' and Lag(coalesce (E.surname,CTE.surname1),2,0) over (order by CTE.audtime) IS NOT NULL then
	coalesce(Lag(coalesce (E.surname,CTE.surname1,'DELETED'),2,0) over (order by CTE.audtime),'ACTIVE1')

	when coalesce (E.surname,CTE.surname1,'DELETED') = 'DELETED' and Lag(coalesce (E.surname,CTE.surname1),3,0) over (order by CTE.audtime) IS NOT NULL then
	coalesce(Lag(coalesce (E.surname,CTE.surname1,'DELETED'),3,0) over (order by CTE.audtime),'ACTIVE1')

	when coalesce (E.surname,CTE.surname1,'DELETED') = 'DELETED' and Lag(coalesce (E.surname,CTE.surname1),4,0) over (order by CTE.audtime) IS NOT NULL then
	coalesce(Lag(coalesce (E.surname,CTE.surname1,'DELETED'),4,0) over (order by CTE.audtime),'ACTIVE1')

	when coalesce (E.surname,CTE.surname1,'DELETED') = 'DELETED' and Lag(coalesce (E.surname,CTE.surname1),5,0) over (order by CTE.audtime) IS NOT NULL then
	coalesce(Lag(coalesce (E.surname,CTE.surname1,'DELETED'),5,0) over (order by CTE.audtime),'ACTIVE1')

	when coalesce (E.surname,CTE.surname1,'DELETED') = 'DELETED' and Lag(coalesce (E.surname,CTE.surname1),6,0) over (order by CTE.audtime) IS NOT NULL then
	coalesce(Lag(coalesce (E.surname,CTE.surname1,'DELETED'),6,0) over (order by CTE.audtime),'ACTIVE1')

	when coalesce (E.surname,CTE.surname1,'DELETED') = 'DELETED' and Lag(coalesce (E.surname,CTE.surname1),7,0) over (order by CTE.audtime) IS NOT NULL then
	coalesce(Lag(coalesce (E.surname,CTE.surname1,'DELETED'),7,0) over (order by CTE.audtime),'ACTIVE1')

	when coalesce (E.surname,CTE.surname1,'DELETED') = 'DELETED' and Lag(coalesce (E.surname,CTE.surname1),8,0) over (order by CTE.audtime) IS NOT NULL then
	coalesce(Lag(coalesce (E.surname,CTE.surname1,'DELETED'),8,0) over (order by CTE.audtime),'ACTIVE1')

	when coalesce (E.surname,CTE.surname1,'DELETED') = 'DELETED' and Lag(coalesce (E.surname,CTE.surname1),9,0) over (order by CTE.audtime) IS NOT NULL then
	coalesce(Lag(coalesce (E.surname,CTE.surname1,'DELETED'),9,0) over (order by CTE.audtime),'ACTIVE1')																
																else '' END As StaffName2

into #AbsenceCheck	
from CTE
left Join Empdetails E on E.Empref = CTE.Empref or (E.forenames = CTE.FirstName and E.surname = CTE.Surname1)


select * from #AbsenceCheck where StaffName2 like @Surname or @Surname = 'All'
order by changedby,date,time
drop table #AbsenceCheck
