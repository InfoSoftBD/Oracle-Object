--------------------------ALL TYPES AND TABLES DROP-----------------------------
drop table property_tab;

drop table person_tab;

drop table appointment_tab;

drop type appointment_list_t force;

drop type property_t force;

drop TYPE appointment_t force;

drop type person_t force;

drop TYPE applicant_t force;

drop type salesperson_t force;

drop type address_t force;

drop TYPE phone_list_t force;

----------------------ALL TYPES AND TABLES CREATE-------------------------------
create type property_t;
/
create type appointment_t;
/
create type appointment_list_t as table of ref appointment_t;
/
Create Type person_t;
/
CREATE TYPE applicant_t;
/
create type salesperson_t;
/
CREATE TYPE address_t;
/

CREATE OR REPLACE TYPE phone_list_t AS VARRAY(10) OF VARCHAR2(20) ;
/

CREATE OR REPLACE TYPE address_t AS OBJECT (
  addressline1   VARCHAR2(200),
  addressline2   VARCHAR2(200),
  town           VARCHAR2(200));
/


create or replace type person_t as object (
person#          VARCHAR2(10),
Surname           VARCHAR2(50),
Forename          varchar2(20),
date_of_birth     DATE,
MEMBER FUNCTION getAge RETURN NUMBER)
not final;
/


CREATE or replace TYPE applicant_t UNDER person_t (
  applicant#     NUMBER,
  maxPrice       NUMBER(10), 
  desiredArea    VARCHAR2(200),
  address        address_t,
  phone          phone_list_t,
  booked         appointment_list_t,
  member function   appointment_booked return number
  );
/

create or replace type salesperson_t under person_t (
  salesperson#      NUMBER, 
  attended          appointment_list_t,
  member function   appointment_attended return number
);
/

create or replace type property_t as object (
  property# char(7), 
  dateOfRegistration    Date, 
  type                  varchar2(20), 
  bedrooms              NUMBER, 
  receptionRooms        NUMBER, 
  bathrooms             NUMBER, 
  garage                VARCHAR2(3), 
  garden                VARCHAR2(20), 
  regionArea            VARCHAR2(200),
  address               address_t,
  price                 NUMBER,
  MEMBER FUNCTION reduced_price RETURN NUMBER
   );
/

CREATE OR REPLACE TYPE BODY person_t AS
  MEMBER FUNCTION getAge RETURN NUMBER AS
  BEGIN
    RETURN Trunc(Months_Between(Sysdate, date_of_birth)/12);
  END getAge;
END;
/

create or replace type body applicant_t as
member function appointment_booked return number is
total number;
  begin
    total := self.booked.count;
    return total;
  end;
end;
/

create or replace type body salesperson_t as
member function appointment_attended return number is
total number;
  begin
    total := self.attended.count;
    return total;
  end;
end;
/

CREATE OR REPLACE TYPE BODY property_t AS
  MEMBER FUNCTION reduced_price RETURN NUMBER AS
  BEGIN
    RETURN (self.price-(self.price*10)/100);
  END reduced_price;
END;
/

create or replace type appointment_t as object (
  appointment#        char(7),
  appdate             DATE, 
  apptime             NUMBER, 
  appointmentType     VARCHAR2(20), 
  levelOfInterest     NUMBER, 
  offerMade           NUMBER,
  is_attended_by ref salesperson_t,
  is_booked_by ref applicant_t,
  property_to_view ref property_t);
/

create table person_tab of person_t (
person# primary key);

create table property_tab of property_t (
property# primary key);

create table appointment_tab of appointment_t (
appointment# primary key,
scope for (is_attended_by) is person_tab,
scope for (is_booked_by) is person_tab,
scope for (property_to_view) is property_tab) ;

-------------------------------SALES PERSON DATA INSERT--------------------------------------------------------------------------------
insert into person_tab
values (salesperson_t('s123', 'Alam', 'Smith', TO_DATE('25/01/1999','DD/MM/YYYY'), 1001, appointment_list_t()));

insert into person_tab
values (salesperson_t('s234', 'Kalam', 'Jones', TO_DATE('17/07/1983','DD/MM/YYYY'), 1002, appointment_list_t()));

insert into person_tab
values (salesperson_t('s345', 'Fandle','Evans', TO_DATE('22/12/1997','DD/MM/YYYY'), 1003, appointment_list_t()));

insert into person_tab
values (salesperson_t('s456', 'Rige', 'Rees', TO_DATE('03/08/1990','DD/MM/YYYY'), 1004, appointment_list_t()));

insert into person_tab
values (salesperson_t('s567', 'Johnson','Morris', TO_DATE('03/07/1982','DD/MM/YYYY'), 1004, appointment_list_t()));

insert into person_tab
values (salesperson_t('s678', 'Fitzel','Farthing', TO_DATE('05/11/1980','DD/MM/YYYY'), 1005, appointment_list_t()));

-------------------------------APPLICANT  DATA INSERT---------------------------------------------------------------------------------
insert into person_tab
values (applicant_t('a123', 'Kishor', 'Das', TO_DATE('25/01/1999','DD/MM/YYYY'), 2002,5000,'Pontry',
        address_t('2 Avocet Drive','Cardiff','CARDIFF'),
        phone_list_t('044-0750-3331398','8888-01711-136873'), appointment_list_t()));
 
insert into person_tab
values (applicant_t('a234', 'Amzad', 'Khan', TO_DATE('05/02/1982','DD/MM/YYYY'), 2002,5000,'Pontry',
        address_t('5 Sun Shine Road','Cardiff','CARDIFF'),
        phone_list_t('044-0750-3331398','8888-01711-136873'), appointment_list_t()));

insert into person_tab
values (applicant_t('a345', 'Saiful', 'Islam', TO_DATE('02/02/1985','DD/MM/YYYY'), 2002,5000,'Pontry',
        address_t('19 Myrther Road','Myrther','Myrther'),
        phone_list_t('044-0750-3331398','8888-01711-136873'), appointment_list_t()));

insert into person_tab
values (applicant_t('a456', 'Faruk', 'Shahin', TO_DATE('20/07/1975','DD/MM/YYYY'), 2002,5000,'Pontry',
        address_t('27 Monk Street','Mountain Ash','Aberdare'),
        phone_list_t('044-0750-3331398','8888-01711-136873'), appointment_list_t()));

insert into person_tab
values (applicant_t('a567', 'Ruhul', 'Islam', TO_DATE('22/05/1989','DD/MM/YYYY'), 2002,5000,'Pontry',
        address_t('20 Stow Hill','Aberdare','ABERDARE'),
        phone_list_t('044-0750-3331398','8888-01711-136873'), appointment_list_t()));

insert into person_tab
values (applicant_t('a678', 'Shaid', 'Khan', TO_DATE('15/03/1998','DD/MM/YYYY'), 2002,5000,'Pontry',
        address_t('3 Marine Drive','Pontypridd','PONTYPRIDD'),
        phone_list_t('044-0750-3331398','8888-01711-136873'), appointment_list_t()));
        
-------------------------------PROPERTY  DATA INSERT---------------------------------------------
insert into property_tab values(
        'PR1001', TO_DATE('07/01/1999','DD/MM/YYYY'), 'DETACHED', 2, 1, 1, 'Y', 'SMALL', 'PONTYPRIDD',
         address_t('2 Avocet Drive','Cardiff','CARDIFF'),5000); 

insert into property_tab values(
        'PR1002', TO_DATE('15/03/2012','DD/MM/YYYY'), 'DETACHED', 2, 1, 1, 'N', 'SMALL', 'NEWPORT',
         address_t('164 Wood Road','Treforest','Pontypridd'),7000);
 
insert into property_tab values(
        'PR1003', TO_DATE('18/06/2020','DD/MM/YYYY'), 'DETACHED', 1, 1, 1, 'Y', 'SMALL', 'GLAMORGAN', 
         address_t('138 Broadway','Treforest','Pontypridd'),3000);

insert into property_tab values(
        'PR1004', TO_DATE('05/01/2021','DD/MM/YYYY'), 'DETACHED', 1, 1, 1, 'N', 'SMALL', 'PONTYPRIDD', 
         address_t('2 Stowhill','Treforest','CARDIFF'),4000); 

insert into property_tab values(
       'PR1005', TO_DATE('08/03/2015','DD/MM/YYYY'), 'DETACHED', 2, 1, 1, 'Y', 'SMALL', 'ABERDARE', 
        address_t('26 Merin Drive','Cardiff','CARDIFF'),6000);

insert into property_tab values(
       'PR1006', TO_DATE('16/01/2010','DD/MM/YYYY'), 'DETACHED', 3, 1, 1, 'N', 'SMALL', 'CARDIF', 
        address_t('Richmond Road','Cardiff','CARDIFF'),7000);
 
-------------------------------APPOINTMENT  DATA INSERT----------------------------------------------------------------

insert into appointment_tab
    select 'APP2001', TO_DATE('07/01/2022','DD/MM/YYYY'), '1600', 'URGENT', 3, 4000, 
    treat(REF(s) as ref salesperson_t),NULL, NULL
from person_tab s
where treat (value(s) as salesperson_t).person#='s123';

insert into appointment_tab
    select 'APP2002', TO_DATE('15/01/2022','DD/MM/YYYY'), '1200', 'URGENT', 3, 3000, 
    treat(REF(s) as ref salesperson_t),NULL, NULL
from person_tab s
where treat (value(s) as salesperson_t).person#='s234';

insert into appointment_tab
select 'APP2003', TO_DATE('21/01/2022','DD/MM/YYYY'),'1500', 'REGULAR', 3, 4000, 
    treat(REF(s) as ref salesperson_t),NULL, NULL
from person_tab s
where treat (value(s) as salesperson_t).person#='s345';

insert into appointment_tab
select 'APP2004', TO_DATE('25/01/2022','DD/MM/YYYY'),'1730', 'URGENT', 3, 5000, 
    treat(REF(s) as ref salesperson_t),NULL, NULL
from person_tab s
where treat (value(s) as salesperson_t).person#='s456';

insert into appointment_tab
select 'APP2005', TO_DATE('27/01/2022','DD/MM/YYYY'), '1100', 'URGENT', 3, 6000, 
    treat(REF(s) as ref salesperson_t),NULL, NULL
from person_tab s
where treat (value(s) as salesperson_t).person#='s567';

insert into appointment_tab
select 'APP2006', TO_DATE('12/01/2022','DD/MM/YYYY'), '1045', 'URGENT', 3, 5000,
    treat(REF(s) as ref salesperson_t),NULL, NULL
from person_tab s
where treat (value(s) as salesperson_t).person#='s678';

-----------------------------Applicant Data update------------------------------
UPDATE appointment_tab SET is_booked_by = 
(SELECT treat(REF(s) as ref applicant_t) FROM person_tab s where treat (value(s) as applicant_t).person#='a123')
WHERE appointment# = 'APP2001';

UPDATE appointment_tab SET is_booked_by = 
(SELECT treat(REF(s) as ref applicant_t) FROM person_tab s where treat (value(s) as applicant_t).person#='a234')
WHERE appointment# = 'APP2002';

UPDATE appointment_tab SET is_booked_by = 
(SELECT treat(REF(s) as ref applicant_t) FROM person_tab s where treat (value(s) as applicant_t).person#='a345')
WHERE appointment# = 'APP2003';

UPDATE appointment_tab SET is_booked_by = 
(SELECT treat(REF(s) as ref applicant_t) FROM person_tab s where treat (value(s) as applicant_t).person#='a456')
WHERE appointment# = 'APP2004';

UPDATE appointment_tab SET is_booked_by = 
(SELECT treat(REF(s) as ref applicant_t) FROM person_tab s where treat (value(s) as applicant_t).person#='a567')
WHERE appointment# = 'APP2005';

UPDATE appointment_tab SET is_booked_by = 
(SELECT treat(REF(s) as ref applicant_t) FROM person_tab s where treat (value(s) as applicant_t).person#='a678')
WHERE appointment# = 'APP2006';

-----------------------------Property Data update------------------------------
UPDATE appointment_tab SET property_to_view = 
(SELECT treat(REF(s) as ref property_t) FROM property_tab s where treat (value(s) as property_t).property#='PR1001')
WHERE appointment# = 'APP2001';

UPDATE appointment_tab SET property_to_view = 
(SELECT treat(REF(s) as ref property_t) FROM property_tab s where treat (value(s) as property_t).property#='PR1002')
WHERE appointment# = 'APP2002';

UPDATE appointment_tab SET property_to_view = 
(SELECT treat(REF(s) as ref property_t) FROM property_tab s where treat (value(s) as property_t).property#='PR1003')
WHERE appointment# = 'APP2003';

UPDATE appointment_tab SET property_to_view = 
(SELECT treat(REF(s) as ref property_t) FROM property_tab s where treat (value(s) as property_t).property#='PR1004')
WHERE appointment# = 'APP2004';

UPDATE appointment_tab SET property_to_view = 
(SELECT treat(REF(s) as ref property_t) FROM property_tab s where treat (value(s) as property_t).property#='PR1005')
WHERE appointment# = 'APP2005';

UPDATE appointment_tab SET property_to_view = 
(SELECT treat(REF(s) as ref property_t) FROM property_tab s where treat (value(s) as property_t).property#='PR1006')
WHERE appointment# = 'APP2006';


-------------------------APPLICANT TABLE DATA-----------------------------------------
insert into TABLE(select treat(value (s) as applicant_t).booked 
from person_tab s
where treat( value(s) as applicant_t).person#='a123') 
select ref(c)
from appointment_tab c
where c.is_booked_by.person#='a123';

insert into TABLE(select treat(value (s) as applicant_t).booked 
from person_tab s
where treat( value(s) as applicant_t).person#='a234') 
select ref(c)
from appointment_tab c
where c.is_booked_by.person#='a234';


insert into TABLE(select treat(value (s) as applicant_t).booked 
from person_tab s
where treat( value(s) as applicant_t).person#='a345') 
select ref(c)
from appointment_tab c
where c.is_booked_by.person#='a345';


insert into TABLE(select treat(value (s) as applicant_t).booked 
from person_tab s
where treat( value(s) as applicant_t).person#='a456') 
select ref(c)
from appointment_tab c
where c.is_booked_by.person#='a456';


insert into TABLE(select treat(value (s) as applicant_t).booked 
from person_tab s
where treat( value(s) as applicant_t).person#='a567') 
select ref(c)
from appointment_tab c
where c.is_booked_by.person#='a567';


insert into TABLE(select treat(value (s) as applicant_t).booked 
from person_tab s
where treat( value(s) as applicant_t).person#='a678') 
select ref(c)
from appointment_tab c
where c.is_booked_by.person#='a678';

-----------------------------------------------------------------------------
insert into TABLE(select treat(value (s) as salesperson_t).attended 
from person_tab s
where treat( value(s) as salesperson_t).person#='s123') 
select ref(c)
from appointment_tab c
where c.is_attended_by.person#='s123';

insert into TABLE(select treat(value (s) as salesperson_t).attended 
from person_tab s
where treat( value(s) as salesperson_t).person#='s234') 
select ref(c)
from appointment_tab c
where c.is_attended_by.person#='s234';

insert into TABLE(select treat(value (s) as salesperson_t).attended 
from person_tab s
where treat( value(s) as salesperson_t).person#='s678') 
select ref(c)
from appointment_tab c
where c.is_attended_by.person#='s678';
-------------------SALES PERSON DATA RETRIVED-----------------------------------

SELECT * FROM person_tab;

SELECT * FROM person_tab p
WHERE VALUE(p) IS OF (applicant_t, salesperson_t) ;

select treat(value(s) as salesperson_t).person# s_number, 
    treat(value(s) as salesperson_t).Forename ForName, 
    treat(value(s) as salesperson_t).Surname SurName, 
    treat(value(s) as salesperson_t).date_of_birth DOB, 
    treat(value(s) as salesperson_t).getAge() Age, 
    value(c).appointment# Appointment#
from person_tab s, table (treat(value(s) as salesperson_t).attended) c;


select treat(value(s) as salesperson_t).person# s_number, 
    treat(value(s) as salesperson_t).Forename ForName, 
    treat(value(s) as salesperson_t).Surname SurName, 
    treat(value(s) as salesperson_t).date_of_birth DOB,
    treat(value(s) as salesperson_t).getAge() Age, 
    treat(value(s) as salesperson_t).appointment_attended() "property attended"
from person_tab s where treat(value(s) as salesperson_t).person# IS NOT NULL;

-------------------APPLICANT DATA RETRIVED--------------------------------------
select treat(value(s) as applicant_t).person# a_number, 
    treat(value(s) as applicant_t).Forename ForName, 
    treat(value(s) as applicant_t).Surname SurName, 
    treat(value(s) as applicant_t).date_of_birth DOB,
    treat(value(s) as applicant_t).getAge() Age,
    treat(value(s) as applicant_t).address.addressline1 Street,
    treat(value(s) as applicant_t).address.addressline2 Area,
    treat(value(s) as applicant_t).address.town Town,
    treat(value(s) as applicant_t).phone Phone,
    treat(value(s) as applicant_t).maxPrice MaxPrice
from person_tab s where treat(value(s) as applicant_t).person# IS NOT NULL;

select treat(value(s) as applicant_t).person# s_number, 
    treat(value(s) as applicant_t).Forename ForName, 
    treat(value(s) as applicant_t).Surname SurName, 
    treat(value(s) as applicant_t).date_of_birth DOB,
    treat(value(s) as applicant_t).getAge() Age,
    treat(value(s) as applicant_t).maxPrice MaximumPrice,
    treat(value(s) as applicant_t).appointment_booked() "property booked"
from person_tab s where treat(value(s) as applicant_t).person# IS NOT NULL;
-------------------------Property Data Retrived---------------------------------
select treat(value(s) as property_t).property# Property#,
    treat(value(s) as property_t).dateOfRegistration RegDate,
    treat(value(s) as property_t).regionArea Area,
    treat(value(s) as property_t). address Address,
    treat(value(s) as property_t).Price Price, 
    treat(value(s) as property_t).reduced_price() ReducedPrice,
    treat(value(s) as property_t).type Type,
    treat(value(s) as property_t).bedrooms Bedrooms,
    treat(value(s) as property_t).receptionRooms Receiption,
    treat(value(s) as property_t).bathrooms Bathrooms,
    treat(value(s) as property_t).garage Garage,
    treat(value(s) as property_t).garden Garden                 
from property_tab s;

-------------------APPOINTMENT DATA RETRIVED--------------------------------------
select treat(value(s) as appointment_t).appointment# Appointment#, 
    treat(value(s) as appointment_t).appdate AppDate, 
    treat(value(s) as appointment_t).apptime AppTime, 
    treat(value(s) as appointment_t).is_attended_by.person# Attendent,
    treat(value(s) as appointment_t).is_booked_by.person# Applicant,
    treat(value(s) as appointment_t).property_to_view.property# Property#
from appointment_tab s where treat(value(s) as appointment_t).appointment# IS NOT NULL;
