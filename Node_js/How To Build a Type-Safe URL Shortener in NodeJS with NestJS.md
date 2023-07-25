# How To Build a Type-Safe URL Shortener in NodeJS with NestJS

```Development``` ```Node.js``` ```TypeScript```

The author selected the Tech Education Fund to receive a donation as part of the Write for DOnations program.


## Introduction


URL, an abbreviation for Uniform Resource Locator, is an address given to a unique resource on the web. Because a URL is unique, no two resources can have the same URL.


The length and complexity of URLs vary. A URL might be as short as example.com or as lengthy as http://llanfairpwllgwyngyllgogerychwyrndrobwllllantysiliogogogoch.co.uk. Complex URLs can be unsightly, cause search engine optimization (SEO) issues, and negatively impact marketing plans. URL shorteners map a long URL to a shorter URL and redirect the user to the original URL when the short URL is used.


In this tutorial, you will create a URL shortener using NestJS. First, you will implement the URL shortening and redirecting logic in a service. Then, you will create route handlers to facilitate the shortening and redirection requests.


# Prerequisites


To follow this tutorial, you will need:


- A local development environment for Node.js version 16 or higher. Follow the How To Install Node.js and Create a Local Development Environment tutorial that suits your system.
- The NestJS CLI installed on your system, which you will set up in Step 1, and familiarity with NestJS. Review Getting Started with NestJS.
- Familiarity with TypeScript. You can review the How To Code in TypeScript series to learn more.

# Step 1 — Preparing Your Development Environment


In this step, you will set up everything you need to start implementing your URL shortening logic. You will install NestJS globally, generate a new NestJS application boilerplate, install dependencies, and create your project’s module, service, and controller.


First, you will install the Nest CLI globally if you have not previously installed it. You will use this CLI to generate your project directory and the required files. Run the following command to install the Nest CLI:


```
npm install -g @nestjs/cli


```


The -g flag will install the Nest CLI globally on your system.


You will see the following output:


```
Output...
added 249 packages, and audited 250 packages in 3m
39 packages are looking for funding
run npm fund for details
found 0 vulnerabilities


```


Then you will use the new command to create the project and generate the necessary boilerplate starter files:


```
nest new URL-shortener


```


You will see the following output:


```
Output...
⚡  We will scaffold your app in a few seconds..

CREATE url-shortener/.eslintrc.js (631 bytes)
CREATE url-shortener/.prettierrc (51 bytes)
CREATE url-shortener/nest-cli.json (118 bytes)
CREATE url-shortener/package.json (2002 bytes)
CREATE url-shortener/README.md (3339 bytes)
CREATE url-shortener/tsconfig.build.json (97 bytes)
CREATE url-shortener/tsconfig.json (546 bytes)
CREATE url-shortener/src/app.controller.spec.ts (617 bytes)
CREATE url-shortener/src/app.controller.ts (274 bytes)
CREATE url-shortener/src/app.module.ts (249 bytes)
CREATE url-shortener/src/app.service.ts (142 bytes)
CREATE url-shortener/src/main.ts (208 bytes)
CREATE url-shortener/test/app.e2e-spec.ts (630 bytes)
CREATE url-shortener/test/jest-e2e.json (183 bytes)

? Which package manager would you ❤️  to use? (Use arrow keys)
> npm
  yarn
  pnpm


```


Choose npm.


You’ll see the following output:


```
Output√ Installation in progress... ☕

🚀  Successfully created project url-shortener
👉  Get started with the following commands:

$ cd url-shortener
$ npm run start

                          Thanks for installing Nest 🙏
                 Please consider donating to our open collective
                        to help us maintain this package.

               🍷  Donate: https://opencollective.com/nest


```


Move to your created project directory:


```
cd url-shortener


```


You will run all subsequent commands in this directory.



Note: The NestJS CLI creates app.controller.ts, app.controller.spec.ts, and app.service.ts files when you generate a new project. Because you won’t need them in this tutorial, you can either delete or ignore them.

Next, you will install the required dependencies.


This tutorial requires a few dependencies, which you will install using NodeJS’s default package manager npm. The required dependencies include TypeORM, SQLite, Class-validator, Class-transformer, and Nano-ID.


TypeORM is an object-relational mapper that facilitates interactions between a TypeScript application and a relational database. This ORM works seamlessly with NestJS due to NestJS’s dedicated @nestjs/typeorm package. You will use this dependency with NestJS’s native typeorm package to interact with an SQLite database.


Run the following command to install TypeORM and its dedicated NestJS package:


```
npm install @nestjs/typeorm typeorm


```


SQLite is a library that implements a small, fast, self-contained SQL database engine. You will use this dependency as your database to store and retrieve shortened URLs.


Run the following command to install SQLite:


```
npm install sqlite3


```


The class-validator package contains decorators used for data validation in NestJS. You will use this dependency with your data-transfer object to validate the data sent into your application.


Run the following command to install class-validator:


```
npm install class-validator


```


The class-transformer package allows you to transform plain objects into an instance of a class and vice-versa. You will use this dependency with the class-validator as it cannot work alone.


Run the following command to install class-transformer:


```
npm install class-transformer


```


Nano-ID is a secure, URL-friendly unique string ID generator. You will use this dependency to generate a unique id for each URL resource.


Run the following command to install Nano-ID:


```
npm install nanoid@^3.0.0


```



Note: Versions of Nano-ID higher than 3.0.0 disabled support for CommonJS modules. This issue could cause an error in your application because the JavaScript code produced by the TypeScript compiler still uses the CommonJS module system.

After you have installed the required dependencies, you will generate the project’s module, service, and controller using the Nest CLI. The module will organize your project, the service will handle all the logic for the URL shortener, and the controller will handle the routes.


Run the following command to generate your module:


```
nest generate module url


```


You will see the following output:


```
OutputCREATE src/url/url.module.ts (80 bytes)
UPDATE src/app.module.ts (304 bytes)

```


Next, run the following command to generate your service:


```
nest generate service url --no-spec


```


The --no-spec flag tells the Nest CLI to generate the files without their test files. You won’t need the test files in this tutorial.


You will see the following output:


```
OutputCREATE src/url/url.service.ts (87 bytes)
UPDATE src/url/url.module.ts (151 bytes)

```


Then run the following command to generate your controller:


```
nest generate controller url --no-spec


```


You will see the following output:


```
OutputCREATE src/url/url.controller.ts (95 bytes)
UPDATE src/url/url.module.ts (233 bytes)

```


In this step, you generated your application and most of the files needed for your development. Next, you’ll connect your application to a database.


# Step 2 — Connecting your Application to a Database


In this step, you’ll create an entity to model the URL resource in your database. An entity is a file containing the necessary properties of the stored data. You will also create a repository as an access layer between your application and its database.


Using nano or your preferred text editor, create and open a file in the src/url folder called url.entity.ts:


```
nano src/url/url.entity.ts


```


This file will contain the entity to model your data.


Next, in your src/url/url.entity.ts file, add the following Typescript code:


src/url/url.entity.ts
```
import { Entity, Column, PrimaryGeneratedColumn } from 'typeorm';

@Entity()
export class Url {
    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    urlCode: string;

    @Column()
    longUrl: string;

    @Column()
    shortUrl: string;
}

```


First, you import the Entity, Column, and PrimaryGeneratedColumn decorators from 'typeorm'.


The code creates and exports a class Url annotated with the Entity decorator that marks a class as an entity.


Each property is specified and annotated with the appropriate decorators: PrimaryGeneratedColumn for the id and Column for the rest of the properties. PrimaryGeneratedColumn is a decorator that automatically generates a value for the properties it annotates. TypeOrm will use it to generate an id for each resource. Column is a decorator that adds a property it annotates as a column in a database.


The properties that should be stored in the database include the following:


- id is the primary key for the database table.
- urlCode is the unique id generated by the nanoid package and will be used to identify each URL.
- longUrl is the URL sent to your application to be shortened.
- shortUrl is the shortened URL.

Save and close the file.


Next, you’ll create a connection between your application and your database.


First, open src/app.module.ts in nano or your preferred text editor:


```
nano src/app.module.ts


```


Then, add the highlighted lines to the file:


src/app.module.ts
```
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { Url } from './url/url.entity';
import { UrlModule } from './url/url.module';

@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'sqlite',
      database: 'URL.sqlite',
      entities: [Url],
      synchronize: true,
    }),
    UrlModule,
  ],
  controllers: [],
  providers: [],
})
export class AppModule {}

```


First, import TypeOrmModule from @nestjs/typeorm and Url from ./url/url.entity. You may still have lines related to AppController and AppService. Leaving them in the file will not affect the rest of the tutorial.


In the imports array, call the forRoot method on the TypeOrmModule to share the connection through all the modules in your application. The forRoot method takes a configuration object as an argument.


The configuration object contains properties that create the connection. These properties include the following:


- The type property denotes the kind of database you are using TypeOrm to interact with. In this case, it is set to 'sqlite'.
- The database property denotes the preferred name for your database. In this case, it is set to 'URL.sqlite'.
- The entities property is an array of all the entities in your project. In this case, you have just one entity specified as Url inside the array.
- The synchronize option automatically syncs your database tables with your entity and updates the tables each time you run the code. In this case, it is set to true.


Note: Setting synchronize to true is only ideal in a development environment. It should always be set to false in production, as it could cause data loss.

Save and close the file.


Next, you’ll create a repository to act as an access layer between your application and your database. You’ll need to connect your entity to its parent module, and this connection enables Nest and TypeOrm to create a repository automatically.


Open src/url/url.module.ts:


```
nano src/url/url.module.ts


```


In the existing file, add the highlighted lines:


src/url/url.module.ts
```
import { Module } from '@nestjs/common';
import { UrlService } from './url.service';
import { UrlController } from './url.controller';
import { TypeOrmModule } from '@nestjs/typeorm';
import { Url } from './url.entity';

@Module({
  imports: [TypeOrmModule.forFeature([Url])],
  providers: [UrlService],
  controllers: [UrlController],
})
export class UrlModule {}

```


You create an imports array inside the Module decorator where you import TypeOrmModule from @nestjs/typeorm and Url from ./url.entity. Inside the imports array, you call the forFeature method on the TypeOrmModule. The forFeature method takes an array of entities as an argument, so you pass in the Url entity.


Save and close the file.


Nest and TypeOrm will create a repository behind the scenes that will act as an access layer between your service and the database.


In this step, you connected your application to a database. You are now ready to implement the URL shortening logic.


# Step 3 — Implementing Service Logic


In this step, you will implement your service logic with two methods. The first method, shortenUrl, will contain all the URL shortening logic. The second method, redirect, will contain all the logic to redirect a user to the original URL. You will also create a data-transfer object to validate the data coming into your application.


## Providing Service Access to the Repository


Before implementing these methods, you will give your service access to your repository to enable your application to read and write data in the database.


First, open src/url/url.service.ts:


```
nano src/url/url.service.ts


```


Add the following highlighted lines to the existing file:


src/url/url.service.ts
```
import { Injectable } from '@nestjs/common';
import { Repository } from 'typeorm';
import { InjectRepository } from '@nestjs/typeorm';
import { Url } from './url.entity';

@Injectable()
export class UrlService {
  constructor(
    @InjectRepository(Url)
    private repo: Repository<Url>,
  ) {}
}

```


You import Repository from typeorm, InjectRepository from @nestjs/typeorm, and Url from ./url.entity.


In your UrlService class, you create a constructor. Inside the constructor, you declare a private variable, repo, as a parameter. Then, you assign a type of Repository to repo with a generic type of Url. You annotate the repo variable with the InjectRepository decorator and pass Url as an argument.


Save and close the file.


Your service now has access to your repository through the repo variable. All the database queries and TypeOrm methods will be called on it.


Next, you will create an asynchronous method, shortenUrl. The method will take a URL as an argument and return a shortened URL. To validate that the data fed into the method is valid, you’ll use a data-transfer object together with the class-validator and class-transformer packages to validate the data.


## Creating a Data-Transfer Object


Before creating the asynchronous method, you will create the data-transfer object required by the shortenUrl async method. A data-transfer object is an object that defines how data will be sent between applications.


First, create a dtos (data-transfer objects) folder within your url folder:


```
mkdir src/url/dtos


```


Then, create a file named url.dto.ts within that folder:


```
nano src/url/dtos/url.dto.ts


```


Add the following code to the new file:


src/url/dto/url.dto.ts
```
import { IsString, IsNotEmpty } from 'class-validator';

export class ShortenURLDto {
  @IsString()
  @IsNotEmpty()
  longUrl: string;
}

```


You import the IsString and IsNotEmpty decorators from class-validator. Then, you create and export the class ShortenURLDto. Inside your ShortenURLDto class, you create a longUrl property and assign it a type of string.


You also annotate the longUrl property with the IsString and IsNotEmpty decorators. Annotating the longUrl property with these decorators will ensure that longUrl is always a string and is not empty.


Save and close the file.


Then, open your src/main.ts file:


```
nano src/main.ts


```


Add the highlighted pieces of code to the existing file:


src/main.ts
```
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { ValidationPipe } from '@nestjs/common';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalPipes(new ValidationPipe({ whitelist: true }));
  await app.listen(3000);
}
bootstrap();

```


You import ValidationPipe, which uses the class-validator package to enforce validation rules on all data coming into your application.


Then, you call the useGlobalPipes method on your application instance (app) and pass an instance of ValidationPipe with an options object where the whitelist property is set to true. The useGlobalPipes method binds ValidationPipe at the application level, ensuring that all routes are protected from incorrect data. Setting the whitelist property to true strips validated (returned) objects of properties that are not specified in your DTO.


Save and close the file.


Next, you’ll import your data-transfer object into the url.service.ts file and apply it to the shortenUrl method.


## Creating the shortenUrl Method


The shortenUrl method will handle most of the URL shortening logic. It will take a parameter url of the type ShortenURLDto.


First, open your src/url/url.service.ts file:


```
nano src/url/url.service.ts


```


Add the highlighted lines to the file:


src/url/url.service.ts
```
import {
  BadRequestException,
  Injectable,
  NotFoundException,
  UnprocessableEntityException,
} from '@nestjs/common';
import { Repository } from 'typeorm';
import { InjectRepository } from '@nestjs/typeorm';
import { Url } from './url.entity';
import { ShortenURLDto } from './dtos/url.dto';
import { nanoid } from 'nanoid';
import { isURL } from 'class-validator';
...

```


First, you import NotFoundExeception, BadRequestException, and UnprocessableEntityException from @nestjs/common because you will use them for error handling. Then, you import {nanoid} from nanoid and isURL from class-validator. isURL will be used to confirm if the supplied longUrl is a valid URL. Finally, you import ShortenURLDto from './dtos/url.dto' for data-validation.


Then, add the following to the UrlService class below the constructor:


src/url/url.service.ts
```
...
async shortenUrl(url: ShortenURLDto) {}

```


Then, add the following code to your shortenUrl method:


src/url/url.service.ts
```
...
    const { longUrl } = url;

    //checks if longurl is a valid URL
    if (!isURL(longUrl)) {
      throw new BadRequestException('String Must be a Valid URL');
    }

    const urlCode = nanoid(10);
    const baseURL = 'http://localhost:3000';

    try {
      //check if the URL has already been shortened
      let url = await this.repo.findOneBy({ longUrl });
      //return it if it exists
      if (url) return url.shortUrl;

      //if it doesn't exist, shorten it
      const shortUrl = `${baseURL}/${urlCode}`;

      //add the new record to the database
      url = this.repo.create({
        urlCode,
        longUrl,
        shortUrl,
      });

      this.repo.save(url);
      return url.shortUrl;
    } catch (error) {
      console.log(error);
      throw new UnprocessableEntityException('Server Error');
    }

```


In the code block above, the longUrl was de-structured from the url object. Then, using the isURL method, a check will validate if longUrl is a valid URL.


A urlCode is generated using nanoid. By default, nanoid generates a unique string of twenty-one characters. Pass your desired length as an argument to override the default behavior. In this case, you pass in a value of 10 because you want the URL to be as short as possible.


Then, the base URL is defined. A base URL is the consistent root of your website’s address. In development, it is your local host server; in production, it is your domain name. For this tutorial, the sample code uses localhost.


A try-catch block will house all the code to interact with the database for error handling.


Shortening a URL twice could lead to duplicate data, so a find query is run on the database to see if the URL exists. If it exists, its shortUrl will be returned; otherwise, the code progresses to shorten it. If no URL record is found in the database, a short URL is created by concatenating the baseURL and the urlCode.


Then, a url entity instance is created with the urlCode, the longUrl, and the shortUrl. The url instance is saved to the database by calling the save method on repo and passing the instance as an argument. Then, the shortUrl is returned.


Finally, if an error occurs, the error is logged to the console in the catch block, and an UnprocessableEntityException message will be thrown.


This is what your url.service.ts file will now look like:


src/url/url.service.ts
```
import {
  BadRequestException,
  Injectable,
  NotFoundException,
  UnprocessableEntityException,
} from '@nestjs/common';
import { Repository } from 'typeorm';
import { InjectRepository } from '@nestjs/typeorm';
import { Url } from './url.entity';
import { ShortenURLDto } from './dtos/url.dto';
import { nanoid } from 'nanoid';
import { isURL } from 'class-validator';

@Injectable()
export class UrlService {
  constructor(
    @InjectRepository(Url)
    private repo: Repository<Url>,
  ) {}

  async shortenUrl(url: ShortenURLDto) {
    const { longUrl } = url;

    //checks if longurl is a valid URL
    if (!isURL(longUrl)) {
      throw new BadRequestException('String Must be a Valid URL');
    }

    const urlCode = nanoid(10);
    const baseURL = 'http://localhost:3000';

    try {
      //check if the URL has already been shortened
      let url = await this.repo.findOneBy({ longUrl });
      //return it if it exists
      if (url) return url.shortUrl;

      //if it doesn't exist, shorten it
      const shortUrl = `${baseURL}/${urlCode}`;

      //add the new record to the database
      url = this.repo.create({
        urlCode,
        longUrl,
        shortUrl,
      });

      this.repo.save(url);
      return url.shortUrl;
    } catch (error) {
      console.log(error);
      throw new UnprocessableEntityException('Server Error');
    }
  }
}

```


Save the file.


Here, you set up the first part of the URL shortening logic. Next, you will implement the redirect method in your service.


## Creating the redirect Method


The redirect method will contain the logic that redirects users to the long URL.


Still in the src/url/url/service.ts file, add the following code to the bottom of your UrlService class to implement the redirect method:


src/url/url.service.ts
```
...
  async redirect(urlCode: string) {
    try {
      const url = await this.repo.findOneBy({ urlCode });
      if (url) return url;
    } catch (error) {
      console.log(error);
      throw new NotFoundException('Resource Not Found');
    }
  }

```


The redirect method takes urlCode as an argument and tries to find a resource in the database with a matching urlCode. If the resource exists, it will return the resource. Else, it throws a NotFoundException error.


Your completed url.service.ts file will now look like this:


src/url/url.service.ts
```
import {
  BadRequestException,
  Injectable,
  NotFoundException,
  UnprocessableEntityException,
} from '@nestjs/common';
import { Repository } from 'typeorm';
import { InjectRepository } from '@nestjs/typeorm';
import { Url } from './url.entity';
import { ShortenURLDto } from './dtos/url.dto';
import { nanoid } from 'nanoid';
import { isURL } from 'class-validator';

@Injectable()
export class UrlService {
  constructor(
    @InjectRepository(Url)
    private repo: Repository<Url>,
  ) {}

  async shortenUrl(url: ShortenURLDto) {
    const { longUrl } = url;

    //checks if longurl is a valid URL
    if (!isURL(longUrl)) {
      throw new BadRequestException('String Must be a Valid URL');
    }

    const urlCode = nanoid(10);
    const baseURL = 'http://localhost:3000';

    try {
      //check if the URL has already been shortened
      let url = await this.repo.findOneBy({ longUrl });
      //return it if it exists
      if (url) return url.shortUrl;

      //if it doesn't exist, shorten it
      const shortUrl = `${baseURL}/${urlCode}`;

      //add the new record to the database
      url = this.repo.create({
        urlCode,
        longUrl,
        shortUrl,
      });

      this.repo.save(url);
      return url.shortUrl;
    } catch (error) {
      console.log(error);
      throw new UnprocessableEntityException('Server Error');
    }
  }

  async redirect(urlCode: string) {
    try {
      const url = await this.repo.findOneBy({ urlCode });
      if (url) return url;
    } catch (error) {
      console.log(error);
      throw new NotFoundException('Resource Not Found');
    }
  }
}

```


Save and close the file.


Your URL shortening logic is now complete with two methods: one to shorten the URL and the other to redirect the shortened URL to the original URL.


In the next step, you will implement the route handler for these two methods in your controller class.


# Step 4 — Implementing Controller Logic


In this step, you’ll create two route handlers: a POST route handler to handle shortening requests and a GET route handler to handle redirection requests.


Before implementing the routes in your controller class, you must make the service available to your controller.


First, open your src/url/url.controller.ts file:


```
nano src/url/url.controller.ts


```


Add the highlighted lines to the file:


src/url/url.controller.ts
```
import { Controller } from '@nestjs/common';
import { UrlService } from './url.service';

@Controller('url')
export class UrlController {
  constructor(private service: UrlService) {}
}

```


First, you import UrlService from ./url.service. Then, in your controller class, you declare a constructor and initialize a private variable, service, as a parameter. You assign service a type of UrlService.


The Controller decorator currently has a string, 'url', as an argument, which means the controller will only handle requests made to localhost/3000/url/route. This behavior introduces a bug in your URL shortening logic as the shortened URLs include a base url (localhost:3000) and a URL code (wyt4_uyP-Il), which combine to form the new URL (localhost:3000/wyt4_uyP-Il). Hence, this controller cannot handle the requests, and the shortened links will return a 404 Not Found error. To resolve this, remove the 'url' argument from the Controller decorator and implement individual routes for each handler.


After you remove the 'url' argument, this is what your UrlController will look like:


src/url/url.controller.ts
```
@Controller()
export class UrlController {
  constructor(private service: UrlService) {}
}

```


Still in the src/url/url.controller.ts file, add the highlighted items to the import statement:


src/url/url.controller.ts
```
import { Body, Controller, Get, Param, Post, Res } from '@nestjs/common';
import { UrlService } from './url.service';
import { ShortenURLDto } from './dtos/url.dto';

```


You import Body, Get, Param, Post, and Res from @nestjs/common and ShortenURLDto from ./dtos/url.dto. The decorators will be further defined as you add to this file.


Then add the following lines to the UrlController below the constructor to define the POST route handler:


src/url/url.controller.ts
```
...
  @Post('shorten')
  shortenUrl(
    @Body()
    url: ShortenURLDto,
  ) {
    return this.service.shortenUrl(url);
  }

```


You create a method shortenUrl that takes an argument of url with a type of ShortenURLDto. You annotate url with the Body decorator to extract the body object from the request object and populate the url variable with its value.


You then annotate the whole method with the Post decorator and pass 'shorten' as an argument. This handler will handle all Post requests made to localhost:<port>/shorten. You then call the shortenUrl method on the service and pass url as an argument.


Next, add the following lines below the POST route to define the GET route handler for redirection:


src/url/url.controller.ts
```
...
  @Get(':code')
  async redirect(
    @Res() res,
    @Param('code')
    code: string,
  ) {
    const url = await this.service.redirect(code);

    return res.redirect(url.longUrl);
  }

```


You create a redirect method with two parameters: res, annotated with the Res decorator, and code, annotated with the Param decorator. The Res decorator turns the class it annotates into an Express response object, allowing you to use library-specific commands like the Express redirect method. The Param decorator extracts the params property from the req object and populates the decorated parameter with its value.


You annotate the redirect method with the Get decorator and pass a wildcard parameter, ':code'. You then pass 'code' as an argument to the Param decorator.


You then call the redirect method on service, await the result, and store it in a variable, url.


Finally, you return res.redirect() and pass url.longUrl as an argument. This method will handle GET requests to localhost:<port>/code, or your shortened URLs.


Your src/url/url.controller.ts file will now look like this:


src/url/url.controller.ts
```
import { Body, Controller, Get, Param, Post, Res } from '@nestjs/common';
import { UrlService } from './url.service';
import { ShortenURLDto } from './dtos/url.dto';

@Controller()
export class UrlController {
  constructor(private service: UrlService) {}

  @Post('shorten')
  shortenUrl(
    @Body()
    url: ShortenURLDto,
  ) {
    return this.service.shortenUrl(url);
  }

  @Get(':code')
  async redirect(
    @Res() res,
    @Param('code')
    code: string,
  ) {
    const url = await this.service.redirect(code);

    return res.redirect(url.longUrl);
  }
}

```


Save and close the file.


Now that you have defined your POST and GET route handlers, your URL shortener is fully functional. In the next step, you will test it.


# Step 5 — Testing the URL Shortener


In this step, you will test the URL shortener you defined in the previous steps.


First, start your application by running:


```
npm run start


```


You will see the following output:


```
Output[Nest] 12640  - 06/08/2022, 16:20:04     LOG [NestFactory] Starting Nest application...
[Nest] 12640  - 06/08/2022, 16:20:07     LOG [InstanceLoader] AppModule dependencies initialized +2942ms
[Nest] 12640  - 06/08/2022, 16:20:07     LOG [InstanceLoader] TypeOrmModule dependencies initialized +1ms
[Nest] 12640  - 06/08/2022, 16:20:08     LOG [InstanceLoader] TypeOrmCoreModule dependencies initialized +257ms
[Nest] 12640  - 06/08/2022, 16:20:08     LOG [InstanceLoader] TypeOrmModule dependencies initialized +2ms
[Nest] 12640  - 06/08/2022, 16:20:08     LOG [InstanceLoader] UrlModule dependencies initialized +4ms
[Nest] 12640  - 06/08/2022, 16:20:08     LOG [RoutesResolver] UrlController {/}: +68ms
[Nest] 12640  - 06/08/2022, 16:20:08     LOG [RouterExplorer] Mapped {/shorten, POST} route +7ms
[Nest] 12640  - 06/08/2022, 16:20:08     LOG [RouterExplorer] Mapped {/:code, GET} route +2ms
[Nest] 12640  - 06/08/2022, 16:20:08     LOG [NestApplication] Nest application successfully started +7ms

```


Open a new terminal to use curl or your preferred API testing tool to make a POST request to http://localhost:3000/shorten with the data below or any data of your choice. For more information on using curl, see the description in this tutorial.


Run this command to make a sample POST request:


```
curl -d "{\"longUrl\":\"http://llanfairpwllgwyngyllgogerychwyrndrobwllllantysiliogogogoch.co.uk\"}" -H "Content-Type: application/json" http://localhost:3000/shorten


```


The -d flag registers the HTTP POST request data, and the -H flag sets the headers for the HTTP request. Running this command will send the long URL to your application and return a shortened URL.


You will receive a short URL as a response, such as the following example:


```
http://localhost:3000/MWBNHDiloW

```


Finally, copy the short URL and paste the link into your browser. Then press ENTER. You will be redirected to the original resource.


# Conclusion


In this article, you created a URL shortener with NestJS. If you add a front-end interface, you can deploy it for public use. You can review the complete project on Github.


NestJS provides the type-safety and architecture to make your application more secure, maintainable, and scalable. To learn more, visit the NestJS official documentation.


