services

services/auth


import { Injectable } from '@angular/core';
import { HttpInterceptor, HttpHandler, HttpRequest, HTTP_INTERCEPTORS } from '@angular/common/http';
import { TokenStorageService } from '../auth/token-storage.service';

const TOKEN_HEADER_KEY = 'Authorization';

@Injectable({
  providedIn: 'root'
})
export class AuthInterceptor implements HttpInterceptor {

  constructor(private tokenStorageService: TokenStorageService) { }

  intercept(req: HttpRequest<any>, next: HttpHandler){
    let authReq = req;
    const token = this.tokenStorageService.getToken();
    if(token != null){
      authReq = req.clone({headers: req.headers.set(TOKEN_HEADER_KEY, 'Bearer '+token)});
    }
    return next.handle(authReq);
  }
}

export const httpInterceptorProviders = [{provide: HTTP_INTERCEPTORS, userClass: AuthInterceptor, multi: true} ];

-------


import { Injectable } from '@angular/core';
import { HttpHeaders, HttpClient } from '@angular/common/http';
import { Observable  } from 'rxjs';
import { AuthLoginInfo } from '../../models/auth/login-info';
import { SignUpInfo } from '../../models/auth/signup-info';
import { JwtResponse } from '../../models/auth/jwt-reponse';

const httpOptions = {
  headers: new HttpHeaders({'Content-Type': 'application/json'})
};

@Injectable({
  providedIn: 'root'
})
export class AuthService {

  private loginUrl = 'http://localhost:8080/api/auth/signin';
  private signupUrl = 'http://localhost:8080/api/auth/signup';

  constructor(private http: HttpClient) { }

  attemptAuth(credentials: AuthLoginInfo): Observable<JwtResponse> {
    return this.http.post<JwtResponse>(this.loginUrl, credentials, httpOptions);
  }

  signUp(info: SignUpInfo): Observable<string>{
    return this.http.post<string>(this.signupUrl, info, httpOptions);
  }
}

---------

import { Injectable } from '@angular/core';

const TOKEN_KEY = 'AuthToken';
const USERNAME_KEY = 'AuthUsername';
const AUTHORITIES_KEY = 'AuthAuthorities';
@Injectable({
  providedIn: 'root'
})
export class TokenStorageService {

  private roles: Array<string> = [];
  constructor() { }

  signOut(){
    window.sessionStorage.clear();
  }

  public saveToken(token: string){
    window.sessionStorage.removeItem(TOKEN_KEY);
    window.sessionStorage.setItem(TOKEN_KEY,token);
  }

  public getToken(): string {
    return window.sessionStorage.getItem(TOKEN_KEY);
  }

  public saveUsername(username: string){
    window.sessionStorage.removeItem(USERNAME_KEY);
    window.sessionStorage.setItem(USERNAME_KEY, username);
  }

  public getUsername():string {
    return sessionStorage.getItem(USERNAME_KEY);
  }

  public saveAuthorities(authorities: string[]) {
    sessionStorage.removeItem(AUTHORITIES_KEY);
    sessionStorage.setItem(AUTHORITIES_KEY, JSON.stringify(authorities));
  }

  public getAuthorities(): string[]{
    this.roles = [];
    if(sessionStorage.getItem(TOKEN_KEY)){
      JSON.parse(sessionStorage.getItem(AUTHORITIES_KEY)).forEach(authority => {
        this.roles.push(authority.authority);
      });
    }
    return this.roles;
  }

}
------------


userservice

import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class UserService {

  private userUrl = 'http://localhost:8080/api/test/user';
  private pmUrl = 'http://localhost:8080/api/test/pm';
  private adminUrl = 'http://localhost:8080/api/test/admin';

  constructor(private http: HttpClient) { }

  getUserBoard(): Observable<string> {
    return this.http.get(this.userUrl, { responseType: 'text' });
  }

  getPMBoard(): Observable<string> {
    return this.http.get(this.pmUrl, { responseType: 'text' });
  }

  getAdminBoard(): Observable<string> {
    return this.http.get(this.adminUrl, { responseType: 'text' });
  }
}


component

admin - ts

import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'app-admin',
  templateUrl: './admin.component.html',
  styleUrls: ['./admin.component.css']
})
export class AdminComponent implements OnInit {

  constructor() { }

  ngOnInit() {
  }

}


admin - html


home - ts

import { Component, OnInit } from '@angular/core';
import { TokenStorageService } from '../services/auth/token-storage.service';

@Component({
  selector: 'app-home',
  templateUrl: './home.component.html',
  styleUrls: ['./home.component.css']
})
export class HomeComponent implements OnInit {

  info: any;

  constructor(private tokenStorageService: TokenStorageService) { }

  ngOnInit() {
    this.info = {
      token: this.tokenStorageService.getToken(),
      username: this.tokenStorageService.getUsername(),
      authorities: this.tokenStorageService.getAuthorities()
    };
  }

  logout() {
    this.tokenStorageService.signOut();
  }

}


home - html

<div *ngIf="info.token; else loggedOut">
  <h5 class="text-primary">Your Info</h5>
  <p>
    <strong>Username </strong>{{info.username}}<br/>
    <strong>Password </strong>{{info.password}}<br/>
    <strong>Username </strong>{{info.username}}<br/>
  </p>
  <button class="btn btn-secondary" type="button" (click)="logout()"></button>
</div>

login - ts

import { Component, OnInit } from '@angular/core';
import { AuthLoginInfo } from '../models/auth/login-info';
import { AuthService } from '../services/auth/auth.service';
import { TokenStorageService } from '../services/auth/token-storage.service';

@Component({
  selector: 'app-login',
  templateUrl: './login.component.html',
  styleUrls: ['./login.component.css']
})
export class LoginComponent implements OnInit {
form: any[];
isLoggedIn = false;
isLoginFailed = false;
errorMessage = '';
roles: string[] = [];
private loginInfo: AuthLoginInfo;
  constructor(private authService: AuthService, private tokenStorageService: TokenStorageService) { }

  ngOnInit() {
    if(this.tokenStorageService.getToken()){
      this.isLoggedIn = true;
      this.roles = this.tokenStorageService.getAuthorities();
    }
  }

  onsubmit(){
    console.log(this.form);
    this.loginInfo = new AuthLoginInfo(this.form.username,this.form.password);
    this.authService.attemptAuth(this.loginInfo).subscribe( data => {
      this.tokenStorageService.saveToken(data.accessToken);
      this.tokenStorageService.saveUsername(data.username);
      this.tokenStorageService.saveAuthorities(data.authorities);
      this.isLoginFailed = false;
      this.isLoggedIn = true;
      this.roles = this.tokenStorageService.getAuthorities();
      this.reloadPage();
    }, error => {
      console.log(error);
      this.errorMessage = error.error.message;
      this.isLoginFailed = true;
    });
  }

  reloadPage(){
    window.location.reload();
  }


}


login - html

<div *ngIf="isLoggedIn; else loggedOut">
  Logged in as {{roles}}.
</div>
<ng-template #loggedOut>
  <div class="row col-sm-6">
    <form name="loginForm" (ngSubmit)="onSubmit()" #loginForm="ngForm" novalidate>
      <div class="form-group">
        <label for="username">Username</label>
        <input type="text" class="form-control" name="username" [(ngModel)]="form.username" #username="ngModel" required/>
        <div *ngIf="loginForm.submitted && username.invalid">
          <div *ngIf="username.errors.required">Username is required</div>
        </div>
      </div>
      <div class="form-group">
        <label for="password">Password</label>
        <input type="password" class="form-control" name="password" [(ngModel)]="form.password" #password="ngModel" required/>
        <div *ngIf="loginForm.submitted && password.invalid">
          <div *ngIf="password.errors.required">Username is required</div>
        </div>
      </div>
      <div class="form-group">
        <button class="btn btn-primary" type="submit">Login</button>
        <div *ngIf="loginForm.submitted && loginFailed" class="alert alert-danger">
          Login failed : {{errorMessage}}
        </div>
      </div>
    </form>
    <hr>
    <p>Don't have an account?</p>
    <a href="signup" class="btn btn-success">Sing Up</a>
  </div>
</ng-template>


pm - ts

pm - html

register - ts

import { Component, OnInit } from '@angular/core';
import { SignUpInfo } from '../models/auth/signup-info';
import { AuthService } from '../services/auth/auth.service';

@Component({
  selector: 'app-register',
  templateUrl: './register.component.html',
  styleUrls: ['./register.component.css']
})
export class RegisterComponent implements OnInit {
  form: any = {};
  signupInfo: SignUpInfo;
  isSignedUp = false;
  isSignedUpFailed = false;
  errorMessage = "";
  constructor(private authService: AuthService) { }

  ngOnInit() {
  }

  onSubmit() {
    console.log(this.form);
    this.signupInfo = new SignUpInfo(this.form.name, this.form.username, this.form.email, this.form.password);
    this.authService.signUp(this.signupInfo).subscribe(data => {
      console.log(data);
      this.isSignedUp = true;
      this.isSignedUpFailed = false;
    }, error => {
      console.log(error);
      this.errorMessage = error.error.message;
      this.isSignedUpFailed = true;
    })
  }

}


register - html

<div *ngIf="isSignedUp; else signupForm">
  Your registration successful. Please login
</div>

<ng-template #signupForm>
  <div class="row col-sm-6">
    <form name="registerForm" #registerForm="ngForm" novalidate>
      <div class="form-group">
        <label for="name">Name</label>
        <input type="text" class="form-control" name="name" [(ngModel)]="form.name" #name="ngModel" required/>
        <div *ngIf="registerForm.submitted && name.invalid">
          <div *ngIf="name.errors.required">Name is required</div>
        </div>
      </div>
      <div class="form-group">
        <label for="name">Username</label>
        <input type="text" class="form-control" name="username" [(ngModel)]="form.username" #username="ngModel" required/>
        <div *ngIf="registerForm.submitted && username.invalid">
          <div *ngIf="username.errors.required">Username is required</div>
        </div>
      </div>
      <div class="form-group">
        <label for="name">Email</label>
        <input type="text" class="form-control" name="email" [(ngModel)]="form.email" #email="ngModel" required/>
        <div *ngIf="registerForm.submitted && email.invalid">
          <div *ngIf="email.errors.required">Email is required</div>
        </div>
      </div>
      <div class="form-group">
        <label for="name">Password</label>
        <input type="text" class="form-control" name="password" [(ngModel)]="form.password" #password="ngModel" required/>
        <div *ngIf="registerForm.submitted && password.invalid">
          <div *ngIf="password.errors.required">Email is required</div>
        </div>
      </div>
      <div class="form-group">
        <button class="btn btn-primary">Register</button>
        <div *ngIf="registeForm.submitted && isSignupFailed" class="alert alert-warning">
          Singup failed!
          <br/>{{errorMessage}}
        </div>
      </div>
    </form>
  </div>
</ng-template>

user - ts

import { Component, OnInit } from '@angular/core';
import { UserService } from '../services/user.service';

@Component({
  selector: 'app-user',
  templateUrl: './user.component.html',
  styleUrls: ['./user.component.css']
})
export class UserComponent implements OnInit {
  board: string;
  errorMessage: string;
  constructor(private userService: UserService) { }

  ngOnInit() {
  }

}


user - html

