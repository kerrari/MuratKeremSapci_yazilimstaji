# MuratKeremSapci_yazilimstaji
Murat Kerem Şapçı-b161200033
--yazilim staji plsql Oracle forms builder triggers and procedures

--PROCEDURES

	PROCEDURE a_kontrol IS
	sayac number;
	sifre_sayac number;

	BEGIN  
	
  	select count(*) into sayac
  	from kerem_yetkili
 	 where kullanici_id = :bl2.kul_adi;
  
  	select count(*) into sifre_sayac
  	from kerem_yetkili
  	where sifre = :bl2.sifre;
   
  	if 
  		sayac = 0 then 
  		mess(1236,'kayýt yok');
  		go_field('bl2.kul_adi');
  
  	elsif 
  		sayac = 1 and sifre_sayac=0 then 
  		mess(1236,'kayit yok');
  		go_field('bl2.sifre');

  	elsif
  		sayac=1 and sifre_sayac=1 then
  		mess(1236,'giris basarili');
  		go_field('uye.hesap_no');

	end if;

	END;

----------------------------------------

	PROCEDURE a_kul_adi_kontrol IS
	v_sayac number ;

	BEGIN

		select count(*) into v_sayac
		from kerem_yetkili 
		where kullanici_id = :bl2.kul_adi;
	
		if v_sayac = 0 then 
		mess(1236,'BOYLE BIR KULLANICI YOKTUR.');
		raise form_trigger_failure;
	
		end if;	   
  
	END;
--------------------------------------------------

	PROCEDURE sifre_kontrol IS
	sifre_sayac number;

	BEGIN  
	
  	select count(*) into sifre_sayac
  	from kerem_yetkili
  	where sifre = :bl2.sifre;
  
 
   
  	if 
  		sifre_sayac = 0 then 
  		mess(1236,'yanlis sifre');
  		go_field('bl2.kul_adi');
  
  	elsif 
  		sifre_sayac = 1 then 
  		go_field('bl2.login');

	end if;

	END;

---------------------------------------
--giden hesapno kontrol procedure
	
	PROCEDURE giden_hesap_no_kontrol IS
	v_alanhesap_no number;

	BEGIN

	if :uye.hav_alan_hesap_no is null then
		mess(1236,'GONDERILEN HESAP NO BOS BIRAKILAMAZ');
		raise form_trigger_failure;
	
	else 
  	select count(*) into v_alanhesap_no
  	from kerem_banka
  	where hesap_no = :uye.hav_alan_hesap_no;
  
  	if v_alanhesap_no=0 then
  		mess(1236,'YANLIS BIR HESAP NUMARASI GIRDINIZ.');
  		raise form_trigger_failure;
  	
  	elsif v_alanhesap_no=1 then
  		select ad_soyad into :uye.alan_adi
		from kerem_yetkili
		where hesap_no = :uye.hav_alan_hesap_no;
		select sube_kodu into :uye.karsi_kod
  	from kerem_banka
  	where hesap_no = :uye.hav_alan_hesap_no;
  	select sube_kategori into :uye.karsi_sube_kat
  	from kerem_banka
  	where hesap_no = :uye.hav_alan_hesap_no;
	
  	go_field('uye.alan_adres');
  	end if;
	end if;
	END;

------------------------------------------------
--gonder procedure
	
	PROCEDURE gonder IS
	v_top_tutar_gon number;
	v_top_tutar_al number;
	BEGIN
		if :uye.masraf_secenek = 'G' then
		
		v_top_tutar_gon := :uye.tutar + :uye.komisyon + :uye.vergi + :uye.masraf;
		
		update kerem_banka
	     set bakiye = bakiye - v_top_tutar_gon
	   where hesap_no = :uye.hesap_no
	   ;
		   
		update kerem_banka
	     set bakiye = bakiye + :uye.tutar
	   where hesap_no = :uye.hav_alan_hesap_no
	   ;   
		   
	elsif :uye.masraf_secenek = 'A' then
		
		v_top_tutar_al := :uye.tutar - :uye.komisyon - :uye.vergi - :uye.masraf;	
		
		update kerem_banka
		set bakiye = bakiye + v_top_tutar_al
		where hesap_no = :uye.hav_alan_hesap_no
		;
		
		update kerem_banka
		set bakiye = bakiye - :uye.tutar
		where hesap_no = :uye.hesap_no
		;
			
		
	end if;
	insert into kerem_giden_havale (hav_ref,hesap_no,bakiye,gonderen_ad,tckn,
				hav_alan_hesap_no,alan_adi,tutar,masraf_secenek,
				masraf_hesapno,masraf,komisyon,vergi)
				values(:uye.hav_ref,:uye.hesap_no,:uye.kul_bakiye,:uye.gonderen_ad,
				:uye.tckn,:uye.hav_alan_hesap_no,:uye.alan_adi,:uye.tutar,
				:uye.masraf_secenek,:uye.masraf_hesapno,:uye.masraf,:uye.komisyon,
				:uye.vergi);
	commit;
	mess(1236,'isleminiz gerceklesmistir.');
	clear_block;
	go_block('bl2');
	clear_block;
	execute_trigger('WHEN-NEW-FORM-INSTANCE');
	END;

--------------------------------

	PROCEDURE hav_ref_al IS
	BEGIN
  	select nvl(max(hav_ref),0)+1 into :uye.hav_ref
  	from kerem_giden_havale;
	END;

-------------------------------------

	PROCEDURE HESAP_NO_KONTROL IS
	v_hesap_no number;
	--v_alanhesap_no number;
	BEGIN
  	select count(*) into v_hesap_no
  	from kerem_banka
  	where hesap_no = :uye.hesap_no;
  
  
  	if v_hesap_no=0 then 
		mess(1236,'YANLIS BIR HESAP NUMARASI GIRDINIZ.');
		raise form_trigger_failure;
  
	  elsif v_hesap_no=1 then
		select bakiye into :uye.kul_bakiye
		from kerem_banka
		where hesap_no=:uye.hesap_no; 
		select ad_soyad into :uye.gonderen_ad
		from kerem_yetkili
		where hesap_no = :uye.hesap_no;
		select SUBE_KODU into :uye.kod
	  from kerem_banka
	  where hesap_no = :uye.hesap_no;
	  select sube_kategori into :uye.sube_kategori
	  from kerem_banka
	  where hesap_no = :uye.hesap_no;
	  end if;
	END;


--------------------------------------
--TRIGGERS
--------------------------------------
--when_new_form_instance trigger
	
	begin 

	SET_WINDOW_PROPERTY('RWINDOW',TITLE,'MURAT KEREM SAPCI STAJ ');
	:bl1.tarih:=To_Tarih(:GLOBAL.sys_date);
	:uye.tarih_pg2:=To_Tarih(:GLOBAL.sys_date);
	hav_ref_al;--prosedur
	go_field('bl2.kul_adi');


	end;
----------------------------------
--bl2.sifre key_next_item_trigger
	
	if length(:bl2.sifre) < 8 then 
	mess(1236,'Sifre 8 karakterden fazla olmalýdýr.');	
		raise form_trigger_failure;
	else
		sifre_kontrol;--prosedur
		go_field('bl2.login');
	end if;
-------------------
--bl2.login when_button_pressed trigger

	if :bl2.kul_adi is null or :bl2.sifre is null then
		mess(1236, 'KULLANICI ADI VEYA SIFRE');
		go_field('bl2.kul_adi');
	else
		a_kontrol;--prosedur
	end if;
---------------------------
--uye.hesap_no key_next_item trigger

	if :uye.hesap_no is null then 
		mess(1236,'HESAP NO BOS OLAMAZ!');
		raise form_trigger_failure ;
		go_field('uye.hesap_no');
	elsif length(:uye.hesap_no)!=8 then
		mess(1236,'HESAP NO 8 HANELI OLMALIDIR.');
		raise form_trigger_failure;
	else 
		hesap_no_kontrol;--prosedur

	go_field('uye.tckn');	
	end if ;
----------------------------------
--uye.tckn key_next_item trigger

	declare 
	v_tc_kontrol number;
	BEGIN

	if length(:uye.tckn) != 11 then
		mess(1236,'TC KIMLIK NO 11 HANELI OLMALIDIR!');
		raise form_trigger_failure ; 
	else 
		select count(*) into v_tc_kontrol
		from kerem_yetkili
		where tckno=:uye.tckn;

		if v_tc_kontrol=0 then
			mess(1236,'hatalý giris yaptýnýz.');
			raise form_trigger_failure;
			--go_field('uye.tckn');
		elsif v_tc_kontrol=1 then
			go_field('uye.adres');
		end if;
	end if ; 
	END;
--------------------------------------------
--uye.adres key_next_item trigger

	if :uye.adres is null then 
		mess(1236,'lutfen adres bilgilerinizi giriniz!');
		raise form_trigger_failure;
		go_field('uye.adres');
	elsif length(:uye.adres)<10 then
		mess(1236,'ADRES BILGILERINIZ 10 KARAKTERDEN AZ OLMAMALIDIR.');
		raise form_trigger_failure;
		go_field('uye.adres');
	else
	go_field('uye.hav_alan_hesap_no');
		end if;
----------------------------------
--uye.hav_alan_hesap_no key_next_item trigger

	if :uye.hav_alan_hesap_no is null then 
		mess(1236,'ALAN HESAP NO BOS OLAMAZ!');
		raise form_trigger_failure ;
		go_field('uye.hav_alan_hesap_no');
	elsif length(:uye.hav_alan_hesap_no)!=8 then
		mess(1236,'ALAN HESAP NO 8 HANELI OLMALIDIR.');
		raise form_trigger_failure;
	else 
	giden_hesap_no_kontrol;--prosedur

	go_field('uye.alan_adres');	
	end if ;
---------------------------------
--uye.alan_adres key_next_item trigger

	if :uye.alan_adres is null then
		mess(1236,'lutfen adres bilgisi giriniz.');
		raise form_trigger_failure;
		go_field('uye.alan_adres');
		elsif length(:uye.alan_adres)<10 then
		mess(1236,'ADRES BILGILERINIZ 10 KARAKTERDEN AZ OLMAMALIDIR.');
		raise form_trigger_failure;
		go_field('uye.alan_adres');
	else
	go_field('uye.aciklama');
	end if;	
--------------------------------------
--uye.aciklama key_next_item trigger

	if :uye.aciklama is null then 
		mess(1236,'LUTFEN BIR ACIKLAMA YAZINIZ!');
		raise form_trigger_failure ;

	elsif length(:uye.aciklama)<5 then
		mess(1236,'ACIKLAMA 5 KARAKTERDEN AZ OLAMAZ.');
		raise form_trigger_failure;
		go_field('uye.aciklama');

	else 
		go_field('uye.tutar');
	end if ;
------------------------------
--uye.tutar key_next_item trigger

	:uye.komisyon := :uye.tutar*3/100;
	:uye.vergi := :uye.tutar*1/100;
	:uye.masraf := (:uye.komisyon + :uye.vergi)*50/100;

	go_field('uye.masraf_secenek');
----------------------------------
--uye.masraf_secenek key_next_item trigger

	if :uye.masraf_secenek is null then
		mess(1236,'LUTFEN SECIM YAPINIZ');
		raise form_trigger_failure;
		go_field('uye.masraf_secenek');
	elsif :uye.masraf_secenek in ('G') then
		:uye.masraf_hesapno := :uye.hesap_no;
	elsif :uye.masraf_secenek in ('A') then
		:uye.masraf_hesapno := :uye.hav_alan_hesap_no;
	end if;

	go_field('uye.gonder');
----------------------------------
--uye.gonder when_button_pressed trigger

	gonder;--(prosedur)
	
	



