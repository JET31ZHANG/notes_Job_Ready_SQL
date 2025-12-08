# 动手练习：创建黑椒唱片商店数据库

目标：
- 检查数据库结构并组织表；
- 使用 SQL 构建数据库；
- 确定数据库中的主要表；
- 创建数据库中的相关表；
- 显示你创建的表及其列。

## 步骤1：检查数据库结构并组织表

开始构建任何数据库前，应按照本课中介绍的步骤对数据结构进行规范化，并创建一个ERD（实体-关系图）或一个包含表和列的列表，以便作为此过程的路线图。

列表形式的数据库结构：

**song**
- **songId(PK)int**
- **songTitle string(100)**
- videoUrl string(100)
- **bandId(FK)int**

**songAlbum**
- **songId(PK,FK1)int**
- **albumId(PK, FK2)int**

**album**
- **albumId(PK)**
- **albumTitle string(100)**
- label string(50)
- releaseDate datetime
- price float(5,2)

**band**
- **bandId(PK)int**
- **bandName string(50)**

**bandArtist**
- **bandId(PK, FK1)int**
- **artistId(PK,FK2)int**

**artist**
- **artist(PK)int**
- artistFirstName string(25)
- **artistLastName string(50)**

加粗列不能为空
额外要求：
- 所有主键列都是自增的整数列
- 在设计步骤中定义的数据类型必须转换为适当的MySQL数据类型。如：
  - 字符串：VARCHAR
  - 浮点数：DECIMAL

### 组织表

先创建主要表（不包括外键的表）
- album
- artist
- band 

相关表（包括依赖其他表格的外键）：
- 歌曲(song)表引用了乐队(band)表；
- 歌曲专辑(songAlbum)表同时引用：歌曲(song)表和专辑(album)表；
- 乐队艺术家(bandArtist)表同时引用：乐队(band)表和艺术家(artist)表。

**相关表也有优先级**
songAlbum 表依赖 song 表，song 表依赖 band 表，故必须先创建 band 表，再创建 song 表。

### 创建脚本文件

vinylrecord shop-schema.sql
- **schema**：数据库的结构，在文件名中包括这个术语表示文件的目的。

## 步骤2：创建数据库

```sql
-- 8-1 vinylrecordshop-schema.sql 包含创建逻辑
/*
 *  Script written by: John Smith
 *  Date written: March 21, 2023
 * */
-- 删除具有相同名称的现有数据库
DROP DATABASE IF EXISTS vinylrecordshop;

-- 创建此数据库
CREATE DATABASE vinylrecordshop;

-- 设置为活动数据库
USE vinylrecordshop;
```

## 步骤3：创建主要表

**album**
- **albumId(PK) int**
- **albumTitle string(100)**
- label string(50)
- releaseDate datetime
- price float(5,2)

```sql
-- 8-2 创建 album 表
/*
 *  albumId列自动递增，如果没有指定值，数据库引擎会自动为每条新纪录分配一个连续的编号，确保每条记录都有不同的主键值；
 *  字符串值使用 VARCHAR 定义，并指定了最大字符数；
 *  releaseDate 是一个 DATE 列。在 MySQL 中，日期默认格式为 yyyy-mm-dd；
 *  price 是一个最大值为 999.99 的 DECIMAL 列，适用于为此解决方案存储的数据；
 *  price、releaseDate 和 label 均是可为空的列；
 *  主键（PRIMARY KEY）约束在 albumId 上定义。
 * */
CREATE TABLE album (
    albumId INT AUTO_INCREMENT,
    albumTitle VARCHAR(100) NOT NULL,
    label VARCHAR(50),
    releaseDate DATE,
    price DECIMAL(5, 2),
    CONSTRAINT pk_album
        PRIMARY KEY (albumId) 
);
```

### 列的顺序

通常将主键放在最前面，有助于提高检索速度。外键放置顺序不重要。CONSTRAINT 定义也可按任意顺序显示，只要约束引用的列先被定义即可。

```sql
-- 8-3 已修改用于创建 album 表的SQL
CREATE TABLE album (
    albumId INT AUTO_INCREMENT,
    CONSTRAINT pk_album
        PRIMARY KEY (albumId),
    albumTitle VARCHAR(100) NOT NULL,
    label VARCHAR(50),
    releaseDate DATE,
    price DECIMAL(5, 2)
);
```

### 动手实践

使用创建 album 表的模式创建 artist 表和 band 表。

**artist**
- **artistId(PK) int**
- artistFirstName string(25)
- **artistLastName string(50)**

**band**
- **bandId(PK) int**
- **bandName string(50)**

使用 DESCRIBE tableName 命令验证表的结构是否正确
使用 DROP TABLE tableName 命令删除现有的表

## 步骤4：创建相关表

数据库中的相关表包括依赖其他表的外键：
- song 表引用 band 表；
- songAlbum 表引用 song 表和 album 表；
- bandArtist 表引用 band 表和 artist 表。

### 创建 song 表

**song**
- **songId(PK) int**
- **songTitle string(100)**
- videoUrl string(100)
- **bandId(FK) int**

前三列包括一个主键（songId）、一个必填列（songTitle）和一个可为空的列（videoUrl）。

```sql
-- 8-4 创建 song 表中的前三列
/*
 *  songId 是自动递增的，不可为空 
 *  songTitle 不可为空，最长 100 个字符
 *  视频URL 最长 100 个字符，该列可选 
 *  songId 作为主键
 * */
CREATE TABLE song (
    songId INT NOT NULL AUTO_INCREMENT,
    songTitle VARCHAR(100) NOT NULL,
    videoUrl VARCHAR(100),
    CONSTRAINT pk_song
        PRIMARY KEY (songId)
);
```

**MySQL 定义外键要完成两个步骤：**
**A. 将该列定义为表中的普通列（将 bandId 添加到 videoUrl 之后）**

```sql
-- 8-5 向表中添加普通列

DROP TABLE IF EXISTS song;
CREATE TABLE song (
    songId INT NOT NULL AUTO_INCREMENT,
    songTitle VARCHAR(100) NOT NULL,
    videoUrl VARCHAR(100),
    bandId INT NOT NULL,
    CONSTRAINT pk_song 
        PRIMARY KEY (songId)
);
```

**B. 在主键约束后添加一个外键约束（在该表的 bandId 列强制执行参照完整性，确保在 song 表中输入的任何值必须先存在于 band 表中。）**

```sql
-- 8-6 添加外键约束的 song 表脚本
DROP TABLE IF EXISTS song;
CREATE TABLE song (
    songId INT NOT NULL AUTO_INCREMENT,
    songTitle VARCHAR(100) NOT NULL,
    videoUrl VARCHAR(100),
    bandId INT NOT NULL,
    CONSTRAINT pk_song 
        PRIMARY KEY (songId),
    CONSTRAINT fk_song_band
        FOREIGN KEY (bandId)
        REFERENCES band(bandId)
);
```

**外键约束使用当前表和主表两个表名，确保约束名称的唯一性。**

### 创建 songAlbum 表

**songAlbum**
- **songId(PK, FK1) INT**
- **albumId(PK, FK2) INT**

注意：
- 该表有一个复合键：主键同时包含两个列；
- 这两个列都是与不同表相关的外键。

首先定义列，但不希望 MySQL 自动编号这些列，只需要将它们定义为整数。

```sql
-- 8-7 初始的 songAlbum 表脚本，其列定义为整数
CREATE TABLE songAlbum (
    songId INT,
    albumId INT
);
```

这些列被指定为不能为空，因它们包括在主键中，实体完整性将强制执行此要求。

添加主键约束：
- 当主键是单个列时，将该列添加到约束中；
- 对于复合键，所有列都列在主键中，用逗号分隔。

```sql
-- 8-8 具有主键约束的 songAlbum 表脚本
DROP TABLE IF EXISTS songAlbum
CREATE TABLE songAlbum (
    songId INT,
    albumId INT,
    CONSTRAINT pk_songAlbum
        PRIMARY KEY (songId, albumId)
);
```

添加两个外键约束

```sql
-- 8-9 完整的 songAlbum 表脚本，包括添加外键约束的部分
DROP TABLE IF EXISTS songAlbum;
CREATE TABLE songAlbum (
    songId INT,
    albumId INT,
    CONSTRAINT pk_songAlbum
        PRIMARY KEY (songId, albumId),
    CONSTRAINT fk_songAlbum_song
        FOREIGN KEY (songId)
        REFERENCES song(songId),
    CONSTRAINT fk_songAlbum_album
        FOREIGN KEY (albumId)
        REFERENCES album(albumId)
);
```

### 自行创建 bandArtist 表

使用创建 songAlbum 表的方法创建 bandArtist 表

**bandArtist**
- **bandId(PK, FK) int**
- **artistId(PK, FK) int**

## 步骤5：完善脚本

```sql
-- 8-10 完整的 vinylrecordshop-schema.sql 脚本
/*
 *  Script written by: John Smith
 *  Date written: March 21, 2023
 * */
-- Running this script will DELETE the existing database and all data 
-- it contains.
-- Use with caution.
DROP DATABASE IF EXISTS vinylrecordshop;
CREATE DATABASE vinylrecordshop;
USE vinylrecordshop;
CREATE TABLE album (
    albumId INT AUTO_INCREMENT,
    albumTitle VARCHAR(100) NOT NULL,
    label VARCHAR(50),
    releaseDate DATE,
    price DECIMAL(5, 2),
    CONSTRAINT pk_album
        PRIMARY KEY (albumId)
);

CREATE TABLE artist (
    artistId INT NOT NULL AUTO_INCREMENT,
    fname VARCHAR(25) NOT NULL,
    lname VARCHAR(50) NOT NULL,
    CONSTRAINT pk_artist
        PRIMARY KEY (artistId)
);

CREATE TABLE band (
    bandId INT AUTO_INCREMENT,
    bandName VARCHAR(50) NOT NULL,
    CONSTRAINT pk_band
        PRIMARY KEY (bandId)
);

CREATE TABLE song (
    songId INT NOT NULL AUTO_INCREMENT,
    songTitle VARCHAR(100) NOT NULL,
    videoUrl VARCHAR(100),
    bandId INT NOT NULL,
    CONSTRAINT pk_song
        PRIMARY KEY (songId),
    CONSTRAINT fk_song_band
        FOREIGN KEY (bandId)
        REFERENCES band(bandId)
);

CREATE TABLE songAlbum (
    songId INT,
    albumId INT,
    CONSTRAINT pk_songAlbum
        PRIMARY KEY (songId, albumId),
    CONSTRAINT fk_songAlbum_song
        FOREIGN KEY (songId)
        REFERENCES song(songId),
    CONSTRAINT fk_songAlbum_album
        FOREIGN KEY (albumId)
        REFERENCES album(albumId)
);

CREATE TABLE bandArtist (
    bandId INT,
    artistId INT,
    CONSTRAINT pk_bandArtist
        PRIMARY KEY (bandId, ArtistId),
    CONSTRAINT fk_bandArtist_band
        FOREIGN KEY (bandId)
        REFERENCES band(bandId),
    CONSTRAINT fk_bandArtist_artist
        FOREIGN KEY (artistId)
        REFERENCES artist(artistId)
);
```

## 小结

- 将 ERD 转换为用于创建黑胶唱片商店数据库及其表的 SQL 代码