# testing
import { ComponentFixture, TestBed } from '@angular/core/testing';
import { LoginComponent, SignUpModel, LoginModel } from './login.component';
import { Router } from '@angular/router';
import { FormsModule } from '@angular/forms';

describe('LoginComponent', () => {
  let component: LoginComponent;
  let fixture: ComponentFixture<LoginComponent>;
  let router: Router;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      declarations: [ LoginComponent ],
      imports: [ FormsModule ],
      providers: [ { provide: Router, useValue: { navigateByUrl: jasmine.createSpy('navigateByUrl') } } ]
    })
    .compileComponents();
  });

  beforeEach(() => {
    fixture = TestBed.createComponent(LoginComponent);
    component = fixture.componentInstance;
    router = TestBed.inject(Router);
    fixture.detectChanges();
  });

  it('should create the component', () => {
    expect(component).toBeTruthy();
  });

  it('should initialize isSignDivVisiable to true', () => {
    expect(component.isSignDivVisiable).toBeTrue();
  });

  it('should initialize signUpObj and loginObj', () => {
    expect(component.signUpObj).toEqual(new SignUpModel());
    expect(component.loginObj).toEqual(new LoginModel());
  });

  it('should register a new user when no users exist', () => {
    spyOn(localStorage, 'setItem');
    component.signUpObj = { name: 'Test User', email: 'test@example.com', password: 'password' };
    component.onRegister();
    expect(localStorage.setItem).toHaveBeenCalledWith('angular17users', JSON.stringify([component.signUpObj]));
  });

  it('should add a new user to the existing users', () => {
    spyOn(localStorage, 'getItem').and.returnValue(JSON.stringify([{ name: 'Existing User', email: 'existing@example.com', password: 'password' }]));
    spyOn(localStorage, 'setItem');
    component.signUpObj = { name: 'New User', email: 'new@example.com', password: 'newpassword' };
    component.onRegister();
    expect(localStorage.setItem).toHaveBeenCalledWith('angular17users', JSON.stringify([
      { name: 'Existing User', email: 'existing@example.com', password: 'password' },
      component.signUpObj
    ]));
  });

  it('should find a user and navigate to /dashboard on login', () => {
    spyOn(localStorage, 'getItem').and.returnValue(JSON.stringify([{ name: 'Existing User', email: 'existing@example.com', password: 'password' }]));
    component.loginObj = { email: 'existing@example.com', password: 'password' };
    component.onLogin();
    expect(router.navigateByUrl).toHaveBeenCalledWith('/dashboard');
  });

  it('should show "No User Found" alert on login if user does not exist', () => {
    spyOn(localStorage, 'getItem').and.returnValue(null);
    component.loginObj = { email: 'nonexistent@example.com', password: 'password' };
    spyOn(window, 'alert');
    component.onLogin();
    expect(window.alert).toHaveBeenCalledWith('No User Found');
  });

  it('should store logged in user data in localStorage', () => {
    spyOn(localStorage, 'getItem').and.returnValue(JSON.stringify([{ name: 'Existing User', email: 'existing@example.com', password: 'password' }]));
    spyOn(localStorage, 'setItem');
    component.loginObj = { email: 'existing@example.com', password: 'password' };
    component.onLogin();
    expect(localStorage.setItem).toHaveBeenCalledWith('loggedUser', JSON.stringify({ name: 'Existing User', email: 'existing@example.com', password: 'password' }));
  });

  it('should toggle isSignDivVisiable when the sign-in button is clicked', () => {
    component.isSignDivVisiable = true;
    fixture.detectChanges();
    const loginButton = fixture.nativeElement.querySelector('#login');
    loginButton.click();
    expect(component.isSignDivVisiable).toBeFalse();
  });

  it('should toggle isSignDivVisiable when the sign-up button is clicked', () => {
    component.isSignDivVisiable = false;
    fixture.detectChanges();
    const registerButton = fixture.nativeElement.querySelector('#register');
    registerButton.click();
    expect(component.isSignDivVisiable).toBeTrue();
  });

  it('should call onRegister when the sign-up button is clicked', () => {
    spyOn(component, 'onRegister');
    const signUpButton = fixture.nativeElement.querySelector('.sign-up button');
    signUpButton.click();
    expect(component.onRegister).toHaveBeenCalled();
  });

  it('should call onLogin when the sign-in button is clicked', () => {
    spyOn(component, 'onLogin');
    const signInButton = fixture.nativeElement.querySelector('.sign-in button');
    signInButton.click();
    expect(component.onLogin).toHaveBeenCalled();
  });

  it('should correctly bind signUpObj to the sign-up form', () => {
    component.signUpObj.name = 'Test Name';
    component.signUpObj.email = 'test@example.com';
    component.signUpObj.password = 'password';
    fixture.detectChanges();
    const nameInput = fixture.nativeElement.querySelector('.sign-up input[name="name"]');
    const emailInput = fixture.nativeElement.querySelector('.sign-up input[name="email"]');
    const passwordInput = fixture.nativeElement.querySelector('.sign-up input[name="password"]');
    expect(nameInput.value).toBe('Test Name');
    expect(emailInput.value).toBe('test@example.com');
    expect(passwordInput.value).toBe('password');
  });

  it('should correctly bind loginObj to the sign-in form', () => {
    component.loginObj.email = 'login@example.com';
    component.loginObj.password = 'password';
    fixture.detectChanges();
    const emailInput = fixture.nativeElement.querySelector('.sign-in input[name="email"]');
    const passwordInput = fixture.nativeElement.querySelector('.sign-in input[name="password"]');
    expect(emailInput.value).toBe('login@example.com');
    expect(passwordInput.value).toBe('password');
  });

  it('should handle JSON parsing errors gracefully during login', () => {
    spyOn(localStorage, 'getItem').and.returnValue('invalid JSON');
    spyOn(window, 'alert');
    component.onLogin();
    expect(window.alert).toHaveBeenCalledWith('No User Found');
  });

  it('should handle JSON parsing errors gracefully during registration', () => {
    spyOn(localStorage, 'getItem').and.returnValue('invalid JSON');
    spyOn(localStorage, 'setItem');
    component.signUpObj = { name: 'New User', email: 'new@example.com', password: 'newpassword' };
    component.onRegister();
    expect(localStorage.setItem).toHaveBeenCalledWith('angular17users', JSON.stringify([component.signUpObj]));
  });

  it('should not register a user with an existing email', () => {
    spyOn(localStorage, 'getItem').and.returnValue(JSON.stringify([{ name: 'Existing User', email: 'existing@example.com', password: 'password' }]));
    spyOn(localStorage, 'setItem');
    component.signUpObj = { name: 'Another User', email: 'existing@example.com', password: 'anotherpassword' };
    component.onRegister();
    expect(localStorage.setItem).not.toHaveBeenCalled();
    spyOn(window, 'alert');
    expect(window.alert).toHaveBeenCalledWith('Email already exists');
  });

  it('should not login a user with incorrect password', () => {
    spyOn(localStorage, 'getItem').and.returnValue(JSON.stringify([{ name: 'Existing User', email: 'existing@example.com', password: 'password' }]));
    component.loginObj = { email: 'existing@example.com', password: 'wrongpassword' };
    spyOn(window, 'alert');
    component.onLogin();
    expect(window.alert).toHaveBeenCalledWith('No User Found');
  });

  it('should update existing user password during registration if the email exists', () => {
    spyOn(localStorage, 'getItem').and.returnValue(JSON.stringify([{ name: 'Existing User', email: 'existing@example.com', password: 'oldpassword' }]));
    spyOn(localStorage, 'setItem');
    component.signUpObj = { name: 'Existing User', email: 'existing@example.com', password: 'newpassword' };
    component.onRegister();
    expect(localStorage.setItem).toHaveBeenCalledWith('angular17users', JSON.stringify([
      { name: 'Existing User', email: 'existing@example.com', password: 'newpassword' }
    ]));
  });

  it('should display error message if registration fields are empty', () => {
    component.signUpObj = { name: '', email: '', password: '' };
    spyOn(window, 'alert');
    component.onRegister();
    expect(window.alert).toHaveBeenCalledWith('Please fill all the fields');
  });

  it('should display error message if login fields are empty', () => {
    component.loginObj = { email: '', password: '' };
    spyOn(window, 'alert');
    component.onLogin();
    expect(window.alert).toHaveBeenCalledWith('Please fill all the fields');
  });
});
