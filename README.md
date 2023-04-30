# GAGN-Verkefni2

### Liður 1
#### Trigger fyrir insert into Registration skipunina.
```
drop trigger if exists Check_Status
delimiter €€
create trigger Check_Status
BEFORE INSERT on Registration
for each row
BEGIN
	DECLARE student_status INT;
    DECLARE status_name VARCHAR(255);
    SELECT studentStatus INTO student_status FROM Students WHERE studentID = NEW.studentID;
    IF student_status NOT IN (1, 7, 8) THEN
        SELECT statusName INTO status_name FROM StudentStatus WHERE ID = student_status;
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = status_name;
    END IF;
end €€
delimiter ;
```

### Liður 2
#### Trigger fyrir update Registration skipunina.
```
delimiter €€
create trigger Check_Status
BEFORE update on Registration
for each row
BEGIN
	DECLARE student_status INT;
    DECLARE status_name VARCHAR(255);
    SELECT studentStatusID INTO student_status FROM Students WHERE studentID = NEW.studentID;
    IF student_status NOT IN (1, 7, 8) THEN
        SELECT studentStatusName INTO status_name FROM StudentStatus WHERE studentStatusID = student_status;
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = status_name;
    END IF;
end €€
delimiter ;
```

### Liður 3
#### Samtals einingar nemanda fundnar með Stored Procedure
#### Eftirfarandi kóði virkar ekki og er enn í vinnslu, skila ekki fjölda eininga sem nemandi hefur klárað.
```
delimiter €€
drop procedure if exists student_credits €€
DELIMITER $$
CREATE PROCEDURE student_credits (
    IN student_id INT,
    OUT full_name VARCHAR(255),
    OUT total_credits INT
)
BEGIN
    DECLARE done INT DEFAULT FALSE;
    DECLARE course_num INT;
    DECLARE grade DECIMAL(4,2);
    DECLARE credits INT;
    DECLARE courses_cursor CURSOR FOR
        SELECT courseNumber, grade FROM Registration WHERE studentID = student_id AND grade >= 5;
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;

    SET full_name = (SELECT CONCAT(firstName, ' ', lastName) FROM Students WHERE studentID = student_id);
    SET total_credits = 0;

    OPEN courses_cursor;
    read_loop: LOOP
        FETCH courses_cursor INTO course_num, grade;
        IF done THEN
            LEAVE read_loop;
        END IF;

        SELECT courseCredits INTO credits FROM Courses WHERE courseNumber = course_num;
        SET total_credits = total_credits + credits;
    END LOOP;

    CLOSE courses_cursor;
END $$
DELIMITER ;

call student_credits(1, @name, @credits);
select @name, @credits;
```
