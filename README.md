# Oracle-orient-objet

create type test2_employe_type as object(num number,nom varchar2(30),age number);
create type test2_employees_type as varray(2) of test2_employe_type;
create type test2_departement_type as object(num number,budget number,employees test2_employees_type);

****************************

create table test2_departement_table of test2_departement_type(num primary key);

****************************

insert into test2_departement_table values(1,100,test2_employees_type(test2_employe_type(1,'ammach',22),test2_employe_type(2,'afrougui',22)));
insert into test2_departement_table values(2,200,test2_employees_type());

declare 
  lesemployees test2_employees_type;
begin
  select d.employees into lesemployees from test2_departement_table d where d.num=2;
  lesemployees.extend(2);                                  // premiere case c'est 1 pas 0     // attention il faut que employees vide
  lesemployees(1):=test2_employe_type(3,'bouazid',21);  
  lesemployees(2):=test2_employe_type(4,'khayou',23);   
  update test2_departement_table set employees=lesemployees where num=2; 
end;

***************************

select d.employees from test2_departement_table d;   //il donne comme resultat le constructeur test2_employees_type
select d.employees.nom from test2_departement_table d;   // ca marche pas
select cursor (select e.nom from table(employees) e) from test2_departement_table;
select d.num,cursor (select e.nom from table(employees) e) from test2_departement_table d;



================================================================================================================================

create type test3_siege_social_type as object(nrue number,rue varchar2(30),ville varchar2(30));
create type test3_compagnie_type as object(numComp number,siege_social test3_siege_social_type,nomComp varchar2(30));

***********************

create table test3_compagnie_table of test3_compagnie_type(numComp primary key);

***********************

insert into test3_compagnie_table values(1,test3_siege_social_type(1,'sidi abbad','marrakech'),'norsys');
insert into test3_compagnie_table values(2,test3_siege_social_type(2,'bab lhad','rabat'),'bridge value');

***********************

select c.siege_social from test3_compagnie_table c;   // il donne le constructeur comme resultat

select c.siege_social.rue from test3_compagnie_table c;   //ca marche

=================================================================================================================================

create type test4_siege_social_type as object(nrue number,rue varchar2(30),ville varchar2(30));
create type test4_compagnie_type as object(numComp number,siege_social ref test4_siege_social_type,nomComp varchar2(30));

***********************

create table test4_siege_social_table of test4_siege_social_type(nrue primary key);
create table test4_compagnie_table of test4_compagnie_type(numComp primary key);

***********************

==> siege social table:
insert into test4_siege_social_table values(1,'sidi abbad','marrakech');
insert into test4_siege_social_table values(2,'bab lhad','rabat');

==> compagnie table
insert into test4_compagnie_table values(1,(select REF(s) from test4_siege_social_table s where s.nrue=1),'norsys');
insert into test4_compagnie_table values(2,(select ref(s) from test4_siege_social_table s where s.nrue=2),'bridge value');

***********************

select c.nomComp,c.siege_social.rue from test4_compagnie_table c;

============================================================================================================================

create type test5_siege_social_type as object(nrue number,rue varchar2(30),ville varchar2(30));
create type  test5_compagnie_type as object(numComp number,siege_social test5_siege_social_type,nomComp varchar2(30));
create type test5_avion_type as object(immatricule varchar2(30),typeAvion varchar2(30),capacite number,compagnieRef ref test5_compagnie_type );

**********************

create table test5_compagnie_table of test5_compagnie_type(numComp primary key);
create table test5_avion_table of test5_avion_type(immatricule primary key);

**********************

insert into test5_compagnie_table values(1,test5_siege_social_type(1,'dawdiyat','marrakech'),'ammach_company');

insert into test5_avion_table(immatricule,typeAvion,capacite) values('a1','bwing',2);
update test5_avion_table  set compagnieRef=(select ref(c) from test5_compagnie_table c where c.numComp=1) where immatricule='a1';

**********************

alter type test5_avion_type add member procedure augmenteCapacite(n number) cascade;   //attention c pas alter table!!!

alter type test5_compagnie_type add member function compteFlotte return number cascade;
alter type test5_compagnie_type add member procedure demenage(a test5_siege_social_type) cascade;
alter type test5_compagnie_type add member procedure acheteAvion(i varchar2) cascade;    //attention au varchar2(30)!!!
alter type test5_compagnie_type add member procedure racheteFlotte(c varchar2) cascade;

alter type test5_compagnie_type add static procedure creer(compagnie test5_compagnie_type) cascade;  //attention pas de member

**********************

create or replace type body test5_avion_type as 

 member procedure augmenteCapacite(n number) is 
 begin
   update test5_avion_table set capacite=n+self.capacite where immatricule=self.immatricule;
 end augmenteCapacite;
end;


===> implementation:
set serveroutput on;
declare
 avion test5_avion_type;
begin
   select value(a) into avion from test5_avion_table a where a.immatricule='a1';
   DBMS_OUTPUT.PUT_LINE(avion.capacite);
   avion.augmenteCapacite(3);
   DBMS_OUTPUT.PUT_LINE(avion.capacite);
end;

**********************

create or replace type body test5_compagnie_type as

 member function compteFlotte return number is
  res number;
 begin
    select count(*) into res from test5_avion_table where compagnieRef.numComp=self.nomComp;
    return res;
 end augmenteCapacite;
 
 static procedure creer(compagnie test5_compagnie_type) is 
 begin
   insert into test5_compagnie_table values(compagnie.nomComp,compagnie.siege_social,compagnie.nomComp);
 end creer;
 
end;


===>implementation:
set serveroutput on;
declare
compagnie test5_compagnie_type;
res number;
begin
  select value(c) into compagnie from test5_compagnie_table c where c.nomComp=1;
  res:=compagnie.compteFlotte();
  DBMS_OUTPUT.PUT_LINE(res);
end;

declare 
begin
 test5_compagnie_type.creer(test5_compagnie_type('2',test5_siege_social_type(7,'bab doukkala','marrakech'),'mycompany'));
end;
