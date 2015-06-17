1.�������Ƽ���mongodʵ��������:
	start_rs0_0.bat		-> 4000 (PRIMARY)
	start_rs0_1.bat		-> 4001 (SECONDARY)
	start_rs0_2.bat		-> 4002 (ARBITER)
	
	start_rs1_0.bat		-> 4003 (PRIMARY)
	start_rs1_1.bat		-> 4004 (SECONDARY)
	start_rs1_2.bat		-> 4005 (ARBITER)

2.��ʼ�����ø��Ƽ�rs0
	//��ʼ��������
	> mongo --port 4000
	> rs.initiate()
	rs0:OTHER> rs.conf()
	
	//���SECONDARY��ARBITER�ڵ�
	rs0:PRIMARY> rs.add("jock-PC:4001")
	rs0:PRIMARY> rs.addArb("jock-PC:4002")
	rs0:PRIMARY> rs.status()

3.��ʼ�����ø��Ƽ�rs1��ͬ�ϣ�
	//��ʼ��������
	> mongo --port 4003
	> rs.initiate()
	rs0:OTHER> rs.conf()
	
	//���SECONDARY��ARBITER�ڵ�
	rs0:PRIMARY> rs.add("jock-PC:4004")
	rs0:PRIMARY> rs.addArb("jock-PC:4005")
	rs0:PRIMARY> rs.status()

4.����3�����÷����������У�

	start_cfg_0.bat		-> 4006
	start_cfg_1.bat		-> 4007
	start_cfg_2.bat		-> 4008

5.����mongos·�ɷ�����������
	start_mongos.bat	-> 4009
	
6.��ӷ�Ƭ����Ⱥ��mongos��Ƭ�����ᱣ�浽���÷�������

	> mongo --port 4009
	mongos> sh.addShard("rs0/jock-PC:4000,jock-PC:4001")
	mongos> sh.addShard("rs1/jock-PC:4003,jock-PC:4004")
	mongos> sh.status()
	
����ԭ��
	����һ��MongoDB��Ⱥ�߼�����Ҫ10̨������9��mongod���̣�һ��mongos���̣�
	Ϊ�˽�ʡ������һЩ���̿��Բ�����ͬһ̨�����ϣ��������ԭ���ǣ�
	��֤һ�����Ƽ��е����нڵ�(PRIMARY, SECONDARY, ARBITER)����̨���÷�����
	�ֿ��ڲ�ͬ��������ϡ�
	
���������
	A,B: ��ʾmongodb���Ƽ�ʵ��
	C: ��ʾ����ʵ��
	M: ��ʾ��������
	
	A1 -> shard1 rs0 PRIMARY   4000
	A2 -> shard1 rs0 SECONDARY 4001
	A3 -> shard1 rs0 ARBITER   4002
	
	B1 -> shard2 rs1 PRIMARY   4003
	B2 -> shard2 rs1 SECONDARY 4004
	B3 -> shard2 rs1 ARBITER   4005
	
	C1 -> cfg1 				   4006
	C2 -> cfg2 				   4007
	C3 -> cfg3 				   4008
	
	
	M1: (A1, B3, C1)
	M2: (A3, B1, C2)
	M3: (A2, B2, C3)
	
	
	Ӧ�÷�������mongos������һ��һ��Ӧ�÷�����ֻ�����Ӽ�Ⱥ�е�һ��mongos
	
	��A,B,C�еĽڵ�ֿ�����ͬ���������ϣ�����һ̨����崻����������Ƽ�ʧ�ܣ��޷��Զ�����ת��
	
	PRIMARY,SECONDARY�ɻ���ת����Ҫ����̺�CPU��
	ARBITER�ڵ�ֻ���ٲã����������ݣ�Ҫ����̺�CPU��
	cfg�ڵ�ֻ�������ݱ��棬Ҫ����̵�
	
	��������������࣬��������ԭ��:
		PRIMARY���ȴ���ARBITER��cfg�ڵ�
		SECONDARY���ȴ���cfg�ڵ�
	
	
	�����4����ͬ���õĻ��������²��𷽰�������Դ�õ��������
	
	M1: (A1, B3, C1)
	M2: (A3, B1)
	M3: (A2, C3)
	M4: (B2, C2)


	
	
