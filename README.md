app.module.ts

import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { Routes,RouterModule } from '@angular/router';
import { HttpClientModule } from '@angular/common/http';
import { FormsModule } from '@angular/forms';
import { CommonModule } from '@angular/common';
import { BrowserAnimationsModule  } from '@angular/platform-browser/animations';
import { ToastrModule } from 'ngx-toastr';

import { AppComponent } from './app.component';
import { FirstComponent } from './first/first.component';
import { SecondComponent } from './second/second.component';
import { UpdateDataService } from './services/updateData.service';
import { BooksListComponent } from './book/books-list/books-list.component';
import { AddBookComponent } from './book/add-book/add-book.component';
import { EditBookComponent } from './book/edit-book/edit-book.component';
import { BookService } from './services/book-service.service';

const routes: Routes = [
{ path: "one", component: FirstComponent},
{ path: "two", component: SecondComponent},
{ path: "books", component: BooksListComponent},
{ path: "addBook", component: AddBookComponent}
];

@NgModule({
  declarations: [
    AppComponent,
    FirstComponent,
    SecondComponent,
    BooksListComponent,
    AddBookComponent,
    EditBookComponent
  ],
  imports: [
    BrowserModule,
    RouterModule.forRoot(routes),
    HttpClientModule,
    FormsModule,
    CommonModule,
    BrowserAnimationsModule,
    ToastrModule.forRoot()
  ],
  providers: [UpdateDataService,BookService],
  bootstrap: [AppComponent]
})
export class AppModule { }

models

export class AppResponse{
    message:string;
    result: boolean;
    data: any;
    dataList: any
    constructor(){

    }
}

export class Book {
    id: number;
    title: string;
    author: string;

    constructor() {

    }
}

service 

import { Injectable } from '@angular/core';
import { BehaviorSubject } from 'rxjs';
@Injectable()
export class UpdateDataService {
private myNumber = new BehaviorSubject<number>(1);
cast = this.myNumber.asObservable();

constructor(){

}

updateNumber(number){
    this.myNumber.next(number);
}
}

import { Injectable } from '@angular/core';
import { HttpClient, HttpHeaders } from '@angular/common/http';
import { Book } from '../models/book.model';
import { AppResponse } from '../models/appResponse.model';

const httpOptions = {
  headers: new HttpHeaders({
    'Content-Type': 'application/json'
  })
};

@Injectable({
  providedIn: 'root'
})
export class BookService {

  BOOK_API_URL: string = "http://localhost:8080/bookapi/";


  constructor(private httpClient: HttpClient) { }


  getBooks() {
    return this.httpClient.get<AppResponse>(this.BOOK_API_URL + "books");
  }

  getBookById(id: number) {

  }

  updateBook(id: number, book: Book) {

  }

  deletBookById(id: number) {
    return this.httpClient.delete<AppResponse>(this.BOOK_API_URL+"delete/"+id,httpOptions);
  }

  saveBook(book: Book) {
    return this.httpClient.post<AppResponse>(this.BOOK_API_URL+"save", book, httpOptions);
  }
}

app component

ts

import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  title = 'App';
}


html

<nav class="navbar navbar-default">
  <div class="container-fluid">
    <div class="navbar-header">
      <a class="navbar-brand active" href="#">Books Managment</a>
    </div>
    <ul class="nav navbar-nav">
      <li><a routerLink="/one">First</a></li>
      <li><a routerLink="/two">Second</a></li>
      <li><a routerLink="/books">Books</a></li>
    </ul>
  </div>
</nav>

<router-outlet></router-outlet>



first

import { Component, OnInit } from '@angular/core';
import { UpdateDataService } from '../services/updateData.service';

@Component({
  selector: 'app-first',
  templateUrl: './first.component.html',
  styleUrls: ['./first.component.css']
})
export class FirstComponent implements OnInit {

  no: number = 0;

  constructor(private updateDataService: UpdateDataService) { 
    setInterval(() => { this.UpdateNewNumber() },1000);
  }

  ngOnInit() {
    this.updateDataService.cast.subscribe(num => this.no = num);
  }
 
  UpdateNewNumber(){
      this.updateDataService.updateNumber(this.no);
      this.no++;
      
  }

}


html

<p>
  first works!
</p>


second/second

import { Component, OnInit } from '@angular/core';
import { UpdateDataService } from '../services/updateData.service';
import  Swal  from 'sweetalert2';

@Component({
  selector: 'app-second',
  templateUrl: './second.component.html',
  styleUrls: ['./second.component.css']
})
export class SecondComponent implements OnInit {

  no: number;

  constructor(private updateDataService: UpdateDataService) { 
    setInterval(() => { this.notify() });
  }

  ngOnInit() {
    this.updateDataService.cast.subscribe(num => this.no = num);
  }

  notify(){
    if(this.no ==10){
      Swal.fire("You have reached 10");
    //   Swal.fire({
    //   title: "Alert",
    //   text: "You have reached 10",
    //   type: 'warning',
    //   showConfirmButton: true,
    //   showCancelButton: true,
    //   confirmButtonText: 'Yes, delete it!',
    //   cancelButtonText: 'No, keep it'
    // }).then((result) => {
    //   if(result.value){
    //     Swal.fire("Success");
    //   }else{
    //     Swal.fire("Fail");
    //   }
    //   console.log(result)
    // });
  }
}
}


html

<p>
  second works!
</p>

{{no}}


book-list

import { Component, OnInit } from '@angular/core';
import { Book } from '../../models/book.model';
import { BookService } from '../../services/book-service.service';
import  Swal  from 'sweetalert2';
import { ToastrService } from 'ngx-toastr';


@Component({
  selector: 'app-books-list',
  templateUrl: './books-list.component.html',
  styleUrls: ['./books-list.component.css']
})
export class BooksListComponent implements OnInit {

  books: any = [];
  constructor(private bookService: BookService,private toastr: ToastrService) { }

  ngOnInit() {
    this.getBooks();
  }

  getBooks(){
    this.bookService.getBooks().subscribe(data => {
      console.log(data.dataList[0]);
      this.books = data.dataList[0];
      console.log("Books -> ",this.books);
  });
  
}

deletBookById(id: number){
  this.bookService.deletBookById(id).subscribe(data => {
    var deleteResponse = data.result;
    if(deleteResponse){
      this.toastr.success("Book with Id "+id+" deleted successfully","Delete Employee");
      this.getBooks();
    }
  });
}
}

html

<div class="container">
  <div class="row">
      <div class="col-sm-2">
        </div>
        <div class="col-sm-8">
            <a routerLink="/addBook"><button type="button" class="btn btn-primary btn-sm">New Book</button></a>
            <br><br>
            <table class="table table-striped table-bordered" id="books_data_table">
                <thead>
                  <tr>
                    <th>ID</th>
                    <th>TITLE</th>
                    <th>AUTHOR</th>
                    <th colspan="3">Actions</th>
                  </tr>
                </thead>
                <tbody>
                  <tr *ngFor="let book of books">
                    <td>{{book.id}}</td>
                    <td>{{book.title}}</td>
                    <td>{{book.author}}</td>
                    <td><button type="button" class="btn btn-success btn-sm" (click)="updateBook(book.id)">Update</button></td>
                    <td><button type="button" class="btn btn-warning btn-sm" (click)="deletBookById(book.id)">Delete</button></td>
                  </tr>
                </tbody>
              </table>
        </div>
        <div class="col-sm-2">
        </div>
  </div>
  
</div>

add-book/add-book

import { Component, OnInit } from '@angular/core';
import { Book } from '../../models/book.model';
import { BookService } from '../../services/book-service.service';

@Component({
  selector: 'app-add-book',
  templateUrl: './add-book.component.html',
  styleUrls: ['./add-book.component.css']
})
export class AddBookComponent implements OnInit {

  book = new Book()

  constructor(private bookService: BookService) { }

  ngOnInit() {
  }

  saveEmployee(){
    this.bookService.saveBook(this.book).subscribe(data => {
      this.book = data.data;
      if(data.result){
        console.log("Saved");
      }
    });
  }
}


html

<div class="container">
  <div class="row">
    <div class="col-sm-2"></div>
    <div class="col-sm-8">
      <div class="panel panel-primary">
        <div class="panel-heading">New Employee</div>
        <div class="panel-body">
          <form #addEmployeeForm="ngForm" (ngSubmit)="saveEmployee()">
            <div class="form-group">
              <label>Title</label>
              <input class="form-control" type="text" name="title" [(ngModel)]="book.title" required/>
            </div>
            <div class="form-group">
              <label>Author</label>
              <input class="form-control" type="text" name="author" [(ngModel)]="book.author" required/>
            </div>
            <div>
              <button class="btn btn-success" type="submit" [disabled]="!addEmployeeForm.form.valid">Save</button>
            </div>
          </form>
        </div>
      </div>
    </div>
    <div class="col-sm-2"></div>
  </div>
</div>






App tier


application.properties

# Database Properties
spring.datasource.username= jagans
spring.datasource.password= Jagan@123
spring.datasource.driver-class-name= com.mysql.jdbc.Driver
spring.datasource.url= jdbc:mysql://202.83.25.105:2217/RESERVATION

#Hibernate Properties

hibernate.show_sql = true
hibernate.hbm2ddl.auto = update

model


package com.aalam.assesment.model;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.Table;

@Entity
@Table(name = "books")
public class Book {

	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Long id;
	private String title;
	private String author;

	public Long getId() {
		return id;
	}

	public void setId(Long id) {
		this.id = id;
	}

	public String getTitle() {
		return title;
	}

	public void setTitle(String title) {
		this.title = title;
	}

	public String getAuthor() {
		return author;
	}

	public void setAuthor(String author) {
		this.author = author;
	}

	public Book(Long id, String title, String author) {
		super();
		this.id = id;
		this.title = title;
		this.author = author;
	}

	public Book() {
		super();
	}

	@Override
	public String toString() {
		return "Book [id=" + id + ", title=" + title + ", author=" + author + "]";
	}
}


controller

package com.aalam.assesment.controller;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.PutMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import com.aalam.assesment.model.Book;
import com.aalam.assesment.response.Response;
import com.aalam.assesment.service.BookService;

@RestController
@RequestMapping("/bookapi")
@CrossOrigin
public class BookController {

	@Autowired
	private BookService bookService;
	
	@GetMapping("/books")
	public ResponseEntity<Response> getBooks(){
		Response response = new Response();
		List<Object> dataList = new ArrayList<>();
		List<Book> books = this.bookService.getBooks();
		dataList = Arrays.asList(books);
		response.setDataList(dataList);
		response.setResult(true);
		response.setMessage("Fetched the books successfully");
		return ResponseEntity.ok(response);
	}
	
	@GetMapping("/book/{id}")
	public ResponseEntity<Response> getBookById(@PathVariable("id") Long id){
		Response response = new Response();
		Book book = this.bookService.getBookById(id);
		response.setResult(true);
		response.setMessage("Fetched the book successfully");
		Object data = (Object)book;
		response.setData(data);
		return ResponseEntity.ok(response);
	}
	
	@PostMapping("/save")
	public ResponseEntity<Response> saveBook(@RequestBody Book book){
		Book savedBook = this.bookService.saveBook(book);
		Object data = (Object) savedBook;
		Response response = new Response();
		response.setData(data);
		response.setMessage("Saved successfully");
		response.setResult(true);
		return ResponseEntity.ok(response);
	}
	
	@PutMapping("/update/{id}")
	public ResponseEntity<Response> updateBook(@PathVariable("id") Long id, @RequestBody Book book){
		Book foundBook = this.bookService.getBookById(id);
			Book savedBook = this.bookService.saveBook(book);
			Object data = (Object) savedBook;
			Response response = new Response();
			response.setData(data);
			response.setMessage("Updated successfully");
			response.setResult(true);
			return ResponseEntity.ok(response);
	}
	
	@DeleteMapping("/delete/{id}")
	public ResponseEntity<Response> deletBookById(@PathVariable("id") Long id){
		this.bookService.deletBookById(id);
		Response response = new Response();
		response.setMessage("Deleted successfully");
		response.setResult(true);
		return ResponseEntity.ok(response);
	}
}

service

package com.aalam.assesment.service;

import java.util.List;

import org.springframework.stereotype.Service;

import com.aalam.assesment.model.Book;

@Service
public interface BookService {
	public List<Book> getBooks();
	public Book getBookById(Long id);
	public Book saveBook(Book book);
	public boolean deletBookById(Long id);
}


service impl

package com.aalam.assesment.service.impl;

import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import com.aalam.assesment.model.Book;
import com.aalam.assesment.repository.BookRepository;
import com.aalam.assesment.service.BookService;

@Service
public class BookServiceImpl implements BookService {

	@Autowired
	private BookRepository bookRepository;

	@Override
	public List<Book> getBooks() {
		return this.bookRepository.findAll();
	}

	@Override
	public Book getBookById(Long id) {

		return this.bookRepository.findOne(id);
	}

	@Override
	public Book saveBook(Book book) {

		return this.bookRepository.save(book);
	}

	@Override
	public boolean deletBookById(Long id) {
		this.bookRepository.delete(id);
		return true;
	}

}

repository
package com.aalam.assesment.repository;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import com.aalam.assesment.model.Book;

@Repository
public interface BookRepository extends JpaRepository<Book, Long> {

}

response

package com.aalam.assesment.response;

import java.util.List;

public class Response {
	private String message;
	private boolean result;
	private Object data;
	private List<Object> dataList;

	public Response() {
		super();
	}

	public Response(String message, boolean result, Object data, List<Object> dataList) {
		super();
		this.message = message;
		this.result = result;
		this.data = data;
		this.dataList = dataList;
	}

	public String getMessage() {
		return message;
	}

	public void setMessage(String message) {
		this.message = message;
	}

	public boolean isResult() {
		return result;
	}

	public void setResult(boolean result) {
		this.result = result;
	}

	public Object getData() {
		return data;
	}

	public void setData(Object data) {
		this.data = data;
	}

	public List<Object> getDataList() {
		return dataList;
	}

	public void setDataList(List<Object> dataList) {
		this.dataList = dataList;
	}

	@Override
	public String toString() {
		return "Response [message=" + message + ", result=" + result + ", data=" + data + ", dataList=" + dataList
				+ "]";
	}

}

queries

create table books(id int not null primary key auto_increment, title varchar(100) not null, author varchar(100) not null);
show tables;
select * from books;







