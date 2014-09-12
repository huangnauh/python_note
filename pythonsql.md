
insert:

	>>> ins = users_table.insert()
	>>> ins = ins.values(name='huang',fullname='libo huang')
	>>> str(ins)
	'INSERT INTO users (name, fullname) VALUES (:name, :fullname)'
	>>>result = conn.execute(ins)

	>>> ins = users_table.insert()
	>>> ins.bind = engine
	>>> ins.execute({'name':'zhe','fullname':'zheyang'},{'name':'ye','fullname':'yekeyi'})

	>>> ins = users_table.insert()
	>>> conn.execute(ins,name='wendy', fullname='Wendy Williams')
	
	>>> conn.execute(addresses.insert(), [
	{'user_id': 1, 'email_address' : 'jack@yahoo.com'},
	{'user_id': 1, 'email_address' : 'jack@msn.com'},
	{'user_id': 2, 'email_address' : 'www@www.org'},
	{'user_id': 2, 'email_address' : 'wendy@aol.com'},
	])

	>>> ins = sa.insert(users_table,values={'name':'ha','fullname':"hahha"})
	
	
select:

	>>> s = sa.select([users_table])
	>>> r = conn.execute(s)
	>>> r
	<sqlalchemy.engine.result.ResultProxy object at 0x026B4290>
	>>> for row in r:
		print(row)

	>>> s = sa.select([users_table.c.name,users_table.c.fullname])


Conjunctions

	s = sa.select([
			(users_table.c.fullname+","+addresses.c.email_address).label('title')
			]).where(
				sa.and_(
					users_table.c.id == addresses.c.user_id,
					users_table.c.name.between(sa.bindparam("a1",type_=sa.String),sa.bindparam("a2",type_=sa.String)),
					sa.or_(
						addresses.c.email_address.like(sa.text("'%'")+sa.bindparam("email1",type_=sa.String)),
						addresses.c.email_address.like(sa.text("'%'")+sa.bindparam("email2",type_=sa.String))
						)
					)
				)

	s = sa.select([
		(users_table.c.fullname+","+addresses.c.email_address).label('title')
		]).where(
			sql.and_(
				users_table.c.id == addresses.c.user_id,
				users_table.c.name.between('a','n'),
				sql.or_(
					addresses.c.email_address.like('%@yahoo.com'),
					addresses.c.email_address.like('%@aol.com')
					)
				)
			)

	>>> s = sa.text(
		"SELECT users.fullname || ',' || addresses.email_address as title "
		"FROM users,addresses "
		"WHERE users.id = addresses.user_id "
		"AND users.name BETWEEN :x AND :y "
		"AND (addresses.email_address LIKE :e1 "
		"OR addresses.email_address LIKE :e2)")
	>>> conn.execute(s,x='a',y='n',e1="%@yahoo.com",e2="%@aol.com")


	>>> s = sa.select([
		"users.fullname || ',' || addresses.email_address AS title "
		]).where(
			sa.and_(
				"users.id = addresses.user_id",
				"users.name BETWEEN :x AND :y",
				"(addresses.email_address LIKE :e1 OR addresses.email_address LIKE :e2)"
				)
			).select_from('users, addresses')
	>>> conn.execute(s,x='a',y='n',e1="%@yahoo.com",e2="%@aol.com").fetchall()


	s = sa.select([(users_table.c.fullname+','+addresses.user_id).label('title')]).where(users_table.c.id == addresses.c.user_id).\
	    where(users_table.c.name.between('a','n')).\
	    where(
		    sa.or_(
			    addresses.c.email_address.like("%@yahoo.com"),
			    addresses.c.email_address.like("%@aol.com")
			    )
		    )


	session.query((User.fullname + ',' + Address.email_address).label('title')).filter(sa.and_(User.id == Address.user_id,User.name.between('a','z'))).all()

为什么？

	>>> session.query(User.fullname).join(Address).all()
	2014-08-01 18:38:52,328 INFO sqlalchemy.engine.base.Engine SELECT users.fullname AS users_fullname 
	FROM users JOIN addresses ON users.id = addresses.user_id
	2014-08-01 18:38:52,379 INFO sqlalchemy.engine.base.Engine ()
	[('Jack Bean',), ('Jack Bean',), ('Wendy Williams',), ('Wendy Williams',)]
	>>> session.query(User).join(Address).all()
	2014-08-01 18:39:04,480 INFO sqlalchemy.engine.base.Engine SELECT users.id AS users_id, users.name AS users_name, users.fullname AS users_fullname, users.password AS users_password 
	FROM users JOIN addresses ON users.id = addresses.user_id
	2014-08-01 18:39:04,549 INFO sqlalchemy.engine.base.Engine ()
	[<User(name='jack',fullname='Jack Bean',password='gjffdd')>, <User(name='wendy',fullname='Wendy Williams',password='foobar')>]



stmt = session.query(sa.func.count(Address.user_id)).filter(Address.user_id == User.id).label('address_count')
>>> stmt
<sqlalchemy.sql.elements.Label object at 0x02D372F0>
>>> session.query(User.name,stmt).all()
2014-08-01 19:36:32,716 INFO sqlalchemy.engine.base.Engine SELECT users.name AS users_name, (SELECT count(addresses.user_id) AS count_1 
FROM addresses 
WHERE addresses.user_id = users.id) AS address_count 
FROM users
2014-08-01 19:36:32,753 INFO sqlalchemy.engine.base.Engine ()
[('wendy', 2), ('mary', 0), ('fred', 0)]


stmt = session.query(Address.user_id,sa.func.count('*').label('address_count')).group_by(Address.user_id).subquery()
session.query(User,stmt.c.address_count).outerjoin(stmt,User.id == stmt.c.user_id).all()

[(<User(name='wendy',fullname='Wendy Williams',password='foobar')>, 2), (<User(name='mary',fullname='Mary Contrary',password='xxg527')>, None), (<User(name='fred',fullname='Fred Flinstone',password='blah')>, None)]