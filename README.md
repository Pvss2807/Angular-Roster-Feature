# Project Overview

# Project Title: Conduit Roster Feature Implementation

This project involves the implementation of a new feature called "Roster" in the Conduit application, which is a backend-powered application that displays user statistics. The task was to create a new endpoint in the backend to fetch user statistics and then display this data in a tabular format on the frontend.

# Table of Contents

# Backend Development
1) UserStatsDto
2) UserController
3) UserService
4) Main Application Configuration
5) Postman API Request

# Frontend Development
1) HTML Structure
2) CSS Styling
3) RosterComponent
4) RosterService
5) RosterModule
6) Roster Routes

   
# Backend Development

# 1. UserStatsDto

The UserStatsDto class is a Data Transfer Object (DTO) that defines the structure of the user statistics data that will be fetched from the backend.

```
export class UserStatsDto {
    username: string;
    totalArticles: number;
    totalFavorites: number;
    firstArticleDate: Date | null;
    }
```


# 2. UserController

The UserController class is responsible for handling HTTP requests related to user operations. The new endpoint /api/roster was added to fetch user statistics.


```
   import { Body, Controller, Delete, Get, HttpException, Param, Post, Put, UsePipes, Logger } from '@nestjs/common';
   import { ValidationPipe } from '../shared/pipes/validation.pipe';
   import { CreateUserDto, LoginUserDto, UpdateUserDto } from './dto';
   import { User } from './user.decorator';
   import { IUserRO } from './user.interface';
   import { UserService } from './user.service';
   import { UserStatsDto } from './dto/user-stats.dto';

   import { ApiBearerAuth, ApiTags } from '@nestjs/swagger';

   @ApiBearerAuth()
  @ApiTags('user')
  @Controller()
  export class UserController {
    private readonly logger = new Logger(UserController.name);
  
    constructor(private readonly userService: UserService) {}
  
    // Other existing methods...
  
    @Get('roster')
    async getRoster(): Promise<UserStatsDto[]> {
      this.logger.log('getRoster endpoint was hit');
      return this.userService.getRosterStats();
    }
 }
```


# 3. UserService

The UserService class handles the business logic. The method getRosterStats was added to fetch user statistics, including the total number of articles, total favorites, and the date of the first article.


```
    import { HttpException, HttpStatus, Injectable, Logger } from '@nestjs/common';
    import { EntityManager, wrap } from '@mikro-orm/core';
    import { User } from './user.entity';
    import { UserStatsDto } from './dto/user-stats.dto';
    import { Article } from '../article/article.entity';
    
    @Injectable()
    export class UserService {
      private readonly logger = new Logger(UserService.name);
    
      constructor(private readonly userRepository: UserRepository, private readonly em: EntityManager) {}
    
      // Other existing methods...
    
      async getRosterStats(): Promise<UserStatsDto[]> {
        this.logger.log('Fetching users for roster');
        const users = await this.em.getRepository(User).findAll(); 
        const rosterStats: UserStatsDto[] = [];
    
        for (const user of users) {
          const articles = await this.em.getRepository(Article).find({ author: user });
          const totalArticles = articles.length;
          const totalFavorites = articles.reduce((sum, article) => sum + article.favoritesCount, 0);
          const firstArticleDate = articles.length > 0 ? articles[0].createdAt : null;
    
          rosterStats.push({
            username: user.username,
            totalArticles,
            totalFavorites,
            firstArticleDate,
          });
        }
    
        return rosterStats;
      }
    }
```


# 4. Main Application Configuration

The main application configuration in main.ts sets up the NestJS application, enabling CORS and defining the API prefix.


```
    import { NestFactory } from '@nestjs/core';
    import { DocumentBuilder, SwaggerModule } from '@nestjs/swagger';
    import { AppModule } from './app.module';
    
    async function bootstrap() {
      const appOptions = { cors: true };
      const app = await NestFactory.create(AppModule, appOptions);
      app.enableCors();
      app.setGlobalPrefix('api');
    
      const options = new DocumentBuilder()
        .setTitle('NestJS Realworld Example App')
        .setDescription('The Realworld API description')
        .setVersion('1.0')
        .addBearerAuth()
        .build();
      const document = SwaggerModule.createDocument(app, options);
      SwaggerModule.setup('/docs', app, document);
    
      await app.listen(3000);
    }
    bootstrap().catch((err) => {
      console.log(err);
    });
```

# 5. Postman API Request

The /api/roster endpoint was tested using Postman. The request was successful, returning the expected JSON data with user statistics.

# Frontend Development

# 1. HTML Structure

The HTML structure for displaying the roster data was created. A table was used to list the username, total articles, total favorites, and the first article date.



```
<h1>Conduit Roster</h1>
    <div class="container">
      <table>
        <thead>
          <tr>
            <th>Username</th>
            <th>Total Articles</th>
            <th>Total Favorites</th>
            <th>First Article Date</th>
          </tr>
        </thead>
        <tbody>
            <tr *ngFor="let user of response">
              <td>{{ user.username }}</td>
              <td>{{ user.totalArticles }}</td>
              <td>{{ user.totalFavorites }}</td>
              <td>{{ user.firstArticleDate }}</td>
            </tr>
        </tbody>
      </table>
    </div>
```

# 2. CSS Styling

The CSS styles for the roster table were defined to make it visually appealing and responsive.



```
.container {
        max-width: 800px;
        margin: 0 auto;
        padding: 20px;
    }
    
    table {
        width: 100%;
        border-collapse: collapse;
        margin-top: 20px;
        box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
    }
    
    thead {
        background-color: #f4f4f4;
    }
    
    th, td {
        padding: 12px 15px;
        text-align: left;
    }
    
    th {
        font-weight: bold;
        font-size: 16px;
        border-bottom: 2px solid #ddd;
    }
    
    td {
        border-bottom: 1px solid #ddd;
    }
    
    tbody tr:hover {
        background-color: #f9f9f9;
    }
    
    tbody tr:nth-child(even) {
        background-color: #f4f4f4;
    }
    
    h1 {
        text-align: center;
    }
```

# 3. RosterComponent

The RosterComponent was created to fetch the roster data from the backend and render it in the HTML table.


```
import { ChangeDetectorRef, Component, OnInit } from '@angular/core';
import { ApiResponse, RosterService } from './roster.service';
import { CommonModule } from '@angular/common';

@Component({
  selector: 'realworld-roster',
  templateUrl: './roster.component.html',
  styleUrls: ['./roster.component.css'],
  providers: [],
  imports: [CommonModule],
  standalone: true,
})
export class RosterComponent implements OnInit {
  response: ApiResponse[] = [];

  constructor(private rosterService: RosterService, private cdr: ChangeDetectorRef) {}

  ngOnInit() {
    this.rosterService.getRoster()
      .subscribe(
        (data) => {
          this.response = data;
          this.cdr.detectChanges(); 
        }, 
        (error) => {
          console.error('Error fetching roster data:', error);
        }
      );
  }
}
```

# 4. RosterService

The RosterService was created to handle the HTTP GET request to fetch the roster data from the backend.


```
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

export interface ApiResponse {
  totalArticles: number;
  username: string;
  totalFavorites: number;
  firstArticleDate: string;
}

@Injectable({
  providedIn: 'root'
})
export class RosterService {
  private apiUrl = 'https://3000-trilogygrou-wsengcondui-p3mawlxiltl.ws-us115.gitpod.io/api/roster';

  constructor(private http: HttpClient) {}

  getRoster(): Observable<ApiResponse[]> {
    return this.http.get<ApiResponse[]>(this.apiUrl);
  }
}
```

# 5. RosterModule

The RosterModule was created to encapsulate the roster-related components and services.

```
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';

@NgModule({
  imports: [CommonModule],
})
export class RosterModule {}
6. Roster Routes
The ROSTER_ROUTES were defined to map the path /roster to the RosterComponent.

typescript
Copy code
// frontend/src/app/roster/roster.routes.ts

import { Route } from '@angular/router';
import { RosterComponent } from './roster.component';

export const ROSTER_ROUTES: Route[] = [
  {
    path: '',
    component: RosterComponent,
  },
];
```

# Final Notes
This project demonstrates the complete cycle of adding a new feature, starting from backend development (creating new endpoints and services) to frontend implementation (displaying data in a user-friendly format). The steps included writing DTOs, controllers, services, and integrating the frontend with appropriate HTML, CSS, and Angular services.

By following this README, other developers can understand the structure of the project, how each component interacts, and how to further extend or modify the functionality.
