# information_and_search_system_for_a_motor_vehicle
Development of an information and search system for a motor vehicle enterprise
<div class="container">
  <div class="selects-wrapper">
    <mat-form-field class="form-select" appearance="fill">
      <mat-label>Водій</mat-label>
      <mat-select [(ngModel)]="selectedDriver">
        <mat-option *ngFor="let item of drivers" [value]="item">
          {{ getDriverFullname(item) }}
        </mat-option>
      </mat-select>
    </mat-form-field>
    <mat-form-field class="form-select" appearance="fill">
      <mat-label>Машина</mat-label>
      <mat-select [(ngModel)]="selectedCar">
        <mat-option *ngFor="let item of cars" [value]="item">
          {{ item.model }}
        </mat-option>
      </mat-select>
    </mat-form-field>
  </div>
  <div class="inputs-wrapper">
    <mat-form-field class="form-filed">
      <mat-label class="form-label">Пункт відправлення</mat-label>
      <input matInput type="text" [(ngModel)]="from" />
    </mat-form-field>
    <mat-form-field class="form-filed">
      <mat-label class="form-label">Пункт призначення</mat-label>
      <input matInput type="text" [(ngModel)]="to" />
    </mat-form-field>
  </div>
  <div class="inputs-wrapper">
    <mat-form-field class="form-field">
      <mat-label class="form-label">Дата відправлення</mat-label>
      <input
        matInput
        [(ngModel)]="startDate"
        [matDatepicker]="picker"
        readonly
      />
      <mat-datepicker-toggle matSuffix [for]="picker"></mat-datepicker-toggle>
      <mat-datepicker #picker></mat-datepicker>
    </mat-form-field>
    <mat-form-field class="form-filed">
      <mat-label class="form-label">Тривалість рейсу</mat-label>
      <input matInput type="text" [(ngModel)]="duration" />
    </mat-form-field>
  </div>
  <button
    class="complete-btn"
    [disabled]="!isValid"
    mat-raised-button
    color="primary"
    (click)="createRun()"
  >
    Готово
  </button>
</div>

Код компоненту MainComponent:
import { Component, OnInit } from '@angular/core';
import { map } from 'rxjs/operators';
import { ApiService } from '../api.service';
import { Car, Driver } from '../domain.type';

@Component({
  selector: 'app-main',
  templateUrl: './main.component.html',
  styleUrls: ['./main.component.scss'],
})
export class MainComponent implements OnInit {
  from: string = '';
  to: string = '';
  startDate: string = '';
  duration: number | null = null;
  selectedCar: Car | null = null;
  selectedDriver: Driver | null = null;
  cars: Car[] = [];
  drivers: Driver[] = [];

  constructor(private apiService: ApiService) {}

  get isValid() {
    return (
      this.from &&
      this.to &&
      this.startDate &&
      this.duration &&
      this.selectedCar &&
      this.selectedDriver
    );
  }

  ngOnInit(): void {
    this.apiService
      .getCars()
      .pipe(map((response) => response.result))
      .subscribe((cars) => (this.cars = cars as Car[]));

    this.apiService
      .getDrivers()
      .pipe(map((response) => response.result))
      .subscribe((drivers) => (this.drivers = drivers as Driver[]));
  }

  getDriverFullname(driver: Driver) {
    return `${driver.first_name} ${driver.last_name}`;
  }

  createRun() {
    if (!this.isValid) {
      return;
    }
    this.apiService
      .createRun({
        from: this.from,
        to: this.to,
        startDate: this.startDate,
        duration: this.duration,
        car: this.selectedCar._id,
        driver: this.selectedDriver._id,
      })
      .subscribe(() => {
        this.from = '';
        this.to = '';
        this.startDate = '';
        this.duration = null;
        this.selectedCar = null;
        this.selectedDriver = null;
      });
  }
}

Шаблон компоненту RunsComponent:
<div class="container">
  <div class="runs">
    <p class="runs-title">Рейси</p>
    <div class="table-container" *ngIf="runs.length">
      <table mat-table [dataSource]="runs" class="mat-elevation-z8 runs-table">
        <ng-container matColumnDef="car">
          <th mat-header-cell *matHeaderCellDef>Машина</th>
          <td mat-cell *matCellDef="let element">
            {{ element.car.model }}
          </td>
        </ng-container>

        <ng-container matColumnDef="driver">
          <th mat-header-cell *matHeaderCellDef>Водій</th>
          <td mat-cell *matCellDef="let element">
            {{ getDriverFullname(element.driver) }}
          </td>
        </ng-container>

        <ng-container matColumnDef="from">
          <th mat-header-cell *matHeaderCellDef>Пункт відправлення</th>
          <td mat-cell *matCellDef="let element">
            {{ element.from }}
          </td>
        </ng-container>

        <ng-container matColumnDef="to">
          <th mat-header-cell *matHeaderCellDef>Пункт призначення</th>
          <td mat-cell *matCellDef="let element">
            {{ element.to }}
          </td>
        </ng-container>

        <ng-container matColumnDef="startDate">
          <th mat-header-cell *matHeaderCellDef>Дата відправлення</th>
          <td mat-cell *matCellDef="let element">
            {{ formatDate(element.startDate) }}
          </td>
        </ng-container>

        <ng-container matColumnDef="duration">
          <th mat-header-cell *matHeaderCellDef>Тривалість рейсу</th>
          <td mat-cell *matCellDef="let element">
            {{ formatDuration(element.duration) }}
          </td>
        </ng-container>

        <tr mat-header-row *matHeaderRowDef="displayedColumns"></tr>
        <tr mat-row *matRowDef="let row; columns: displayedColumns"></tr>
      </table>
      <mat-paginator
        [length]="runCount"
        [pageSize]="limit"
        [pageSizeOptions]="[5, 10, 25, 100]"
        aria-label="Виберіть сторінку"
        (page)="onChangePage($event)"
      >
      </mat-paginator>
    </div>
  </div>
</div>

Код компоненту RunsComponent:
import { Component, OnInit, ViewChild } from '@angular/core';
import { map, filter } from 'rxjs/operators';
import { MatTable } from '@angular/material/table';
import { ApiService } from '../api.service';
import { PageEvent } from '@angular/material/paginator';
import { Driver, PopulatedRun } from '../domain.type';

@Component({
  selector: 'app-runs',
  templateUrl: './runs.component.html',
  styleUrls: ['./runs.component.scss'],
})
export class RunsComponent implements OnInit {
  runs: PopulatedRun[] = [];
  page: number = 0;
  limit: number = 5;
  runCount: number = 100;
  displayedColumns: string[] = [
    'car',
    'driver',
    'from',
    'to',
    'startDate',
    'duration',
  ];
  @ViewChild(MatTable, { static: false }) table!: MatTable<PopulatedRun>;
  constructor(private apiService: ApiService) {}

  ngOnInit(): void {
    this.apiService
      .getRunCount()
      .pipe(
        filter((response) => !!response.result),
        map((response) => response.result as number)
      )
      .subscribe((count) => (this.runCount = count));
    this.apiService
      .getRuns(this.page * this.limit, this.limit)
      .pipe(
        filter((response) => !!response.result),
        map((response) => response.result as PopulatedRun[])
      )
      .subscribe((runs) => (this.runs = runs));
  }

  getDriverFullname(driver: Driver) {
    return `${driver.first_name} ${driver.last_name}`;
  }

  formatDate(dateStr: string) {
    const date = new Date(dateStr);
    return `${date.getDate() < 10 ? `0${date.getDate()}` : date.getDate()}.${
      date.getMonth() + 1 < 10 ? `0${date.getMonth() + 1}` : date.getMonth() + 1
    }.${date.getFullYear()} ${
      date.getHours() < 10 ? `0${date.getHours()}` : date.getHours()
    }:${date.getMinutes() < 10 ? `0${date.getMinutes()}` : date.getMinutes()}`;
  }

  formatDuration(minutes: number) {
    const hours = (minutes - (minutes % 60)) / 60;
    return `${hours ? `${hours} годин` : ''}${
      minutes % 60 ? ` ${minutes % 60} хвилин` : ''
    }`;
  }

  onChangePage(event: PageEvent) {
    this.apiService
      .getRuns(event.pageIndex * event.pageSize, event.pageSize)
      .pipe(
        filter((response) => !!response.result),
        map((response) => response.result as PopulatedRun[])
      )
      .subscribe((runs) => (this.runs = runs));
  }
}

Шаблон компоненту CarsComponent:
<div class="container">
  <div class="cars">
    <p class="cars-title">Водії</p>
    <div class="table-container" *ngIf="cars.length">
      <table mat-table [dataSource]="cars" class="mat-elevation-z8 cars-table">
        <ng-container matColumnDef="id">
          <th mat-header-cell *matHeaderCellDef>ID</th>
          <td mat-cell *matCellDef="let element">
            {{ element._id }}
          </td>
        </ng-container>

        <ng-container matColumnDef="model">
          <th mat-header-cell *matHeaderCellDef>Модель</th>
          <td mat-cell *matCellDef="let element">
            {{ element.model }}
          </td>
        </ng-container>

        <tr mat-header-row *matHeaderRowDef="displayedColumns"></tr>
        <tr mat-row *matRowDef="let row; columns: displayedColumns"></tr>
      </table>
      <mat-paginator
        class="paginator"
        [length]="carCount"
        [pageSize]="limit"
        [pageSizeOptions]="[5, 10, 25, 100]"
        aria-label="Виберіть сторінку"
        (page)="onChangePage($event)"
      >
      </mat-paginator>
    </div>
  </div>
</div>

Код компоненту CarsComponent:
import { Component, OnInit, ViewChild } from '@angular/core';
import { map, filter } from 'rxjs/operators';
import { MatTable } from '@angular/material/table';
import { ApiService } from '../api.service';
import { PageEvent } from '@angular/material/paginator';
import { Car } from '../domain.type';

@Component({
  selector: 'app-cars',
  templateUrl: './cars.component.html',
  styleUrls: ['./cars.component.scss'],
})
export class CarsComponent implements OnInit {
  cars: Car[] = [];
  page: number = 0;
  limit: number = 5;
  carCount: number = 100;
  displayedColumns: string[] = ['id', 'model'];
  @ViewChild(MatTable, { static: false }) table!: MatTable<Car>;
  constructor(private apiService: ApiService) {}

  ngOnInit(): void {
    this.apiService
      .getCarCount()
      .pipe(
        filter((response) => !!response.result),
        map((response) => response.result as number)
      )
      .subscribe((count) => (this.carCount = count));
    this.apiService
      .getCars(this.page * this.limit, this.limit)
      .pipe(
        filter((response) => !!response.result),
        map((response) => response.result as Car[])
      )
      .subscribe((cars) => (this.cars = cars));
  }

  onChangePage(event: PageEvent) {
    this.apiService
      .getCars(event.pageIndex * event.pageSize, event.pageSize)
      .pipe(
        filter((response) => !!response.result),
        map((response) => response.result as Car[])
      )
      .subscribe((cars) => (this.cars = cars));
  }
}

Шаблон компоненту DriversComponent:
<div class="container">
  <div class="drivers">
    <p class="drivers-title">Водії</p>
    <div class="table-container" *ngIf="drivers.length">
      <table
        mat-table
        [dataSource]="drivers"
        class="mat-elevation-z8 drivers-table"
      >
        <ng-container matColumnDef="id">
          <th mat-header-cell *matHeaderCellDef>ID</th>
          <td mat-cell *matCellDef="let element">
            {{ element._id }}
          </td>
        </ng-container>

        <ng-container matColumnDef="firstName">
          <th mat-header-cell *matHeaderCellDef>Ім'я</th>
          <td mat-cell *matCellDef="let element">
            {{ element.first_name }}
          </td>
        </ng-container>

        <ng-container matColumnDef="lastName">
          <th mat-header-cell *matHeaderCellDef>Прізвище</th>
          <td mat-cell *matCellDef="let element">
            {{ element.last_name }}
          </td>
        </ng-container>

        <tr mat-header-row *matHeaderRowDef="displayedColumns"></tr>
        <tr mat-row *matRowDef="let row; columns: displayedColumns"></tr>
      </table>
      <mat-paginator
        class="paginator"
        [length]="driverCount"
        [pageSize]="limit"
        [pageSizeOptions]="[5, 10, 25, 100]"
        aria-label="Виберіть сторінку"
        (page)="onChangePage($event)"
      >
      </mat-paginator>
    </div>
  </div>
</div>

Код компоненту DriversComponent:
import { Component, OnInit, ViewChild } from '@angular/core';
import { map, filter } from 'rxjs/operators';
import { MatTable } from '@angular/material/table';
import { ApiService } from '../api.service';
import { PageEvent } from '@angular/material/paginator';
import { Driver } from '../domain.type';

@Component({
  selector: 'app-drivers',
  templateUrl: './drivers.component.html',
  styleUrls: ['./drivers.component.scss'],
})
export class DriversComponent implements OnInit {
  drivers: Driver[] = [];
  page: number = 0;
  limit: number = 5;
  driverCount: number = 100;
  displayedColumns: string[] = ['id', 'firstName', 'lastName'];
  @ViewChild(MatTable, { static: false }) table!: MatTable<Driver>;
  constructor(private apiService: ApiService) {}

  ngOnInit(): void {
    this.apiService
      .getDriverCount()
      .pipe(
        filter((response) => !!response.result),
        map((response) => response.result as number)
      )
      .subscribe((count) => (this.driverCount = count));
    this.apiService
      .getDrivers(this.page * this.limit, this.limit)
      .pipe(
        filter((response) => !!response.result),
        map((response) => response.result as Driver[])
      )
      .subscribe((drivers) => (this.drivers = drivers));
  }

  onChangePage(event: PageEvent) {
    this.apiService
      .getDrivers(event.pageIndex * event.pageSize, event.pageSize)
      .pipe(
        filter((response) => !!response.result),
        map((response) => response.result as Driver[])
      )
      .subscribe((drivers) => (this.drivers = drivers));
  }
}

Код сервісу ApiService:
import { HttpClient } from '@angular/common/http';
import { Injectable } from '@angular/core';
import { Car, Driver, Run, PopulatedRun } from './domain.type';

type ApiResponse<T> = {
  result?: T;
  error?: {
    message: string;
  };
};

@Injectable({
  providedIn: 'root',
})
export class ApiService {
  private url = 'http://127.0.0.1:3001/api';

  constructor(private http: HttpClient) {}

  private callApi<T>(method: string, params: object = {}) {
    return this.http.post<ApiResponse<T>>(this.url, { method, params });
  }

  getRuns(offset = 0, limit = 10) {
    return this.callApi<PopulatedRun[]>('run.find', { offset, limit });
  }

  getCars(offset?: number, limit?: number) {
    return this.callApi<Car[]>('car.find', { offset, limit });
  }

  getDrivers(offset?: number, limit?: number) {
    return this.callApi<Driver[]>('driver.find', { offset, limit });
  }

  createRun(run: Run) {
    return this.callApi<Run>('run.create', { run });
  }

  getRunCount() {
    return this.callApi<number>('run.count', {});
  }

  getCarCount() {
    return this.callApi<number>('car.count', {});
  }

  getDriverCount() {
    return this.callApi<number>('driver.count', {});
  }
}

Код обробників на серверній частині:
const CarRepository = require('../repositories/car')

const carRepository = new CarRepository()

module.exports = {
  'car.find': async ({ offset, limit }) => {
    const result = await carRepository.find(offset, limit)
    return { result }
  },

  'car.count': async () => {
    const result = await carRepository.count()
    return { result }
  },
}

const DriverRepository = require('../repositories/driver')

const driverRepository = new DriverRepository()

module.exports = {
  'driver.find': async ({ offset, limit }) => {
    const result = await driverRepository.find(offset, limit)
    return { result }
  },

  'driver.count': async () => {
    const result = await driverRepository.count()
    return { result }
  },
}

const RunRepository = require('../repositories/run')
const CarRepository = require('../repositories/car')
const DriverRepository = require('../repositories/driver')

const runRepository = new RunRepository()
const carRepository = new CarRepository()
const driverRepository = new DriverRepository()

module.exports = {
  'run.create': async ({ run }) => {
    return runRepository.create(run)
  },

  'run.find': async ({ offset, limit }) => {
    const runs = await runRepository.find(offset, limit)
    const cars = await carRepository.getByRunIds(runs.map(({ car }) => car))
    const drivers = await driverRepository.getByRunIds(runs.map(({ driver }) => driver))

    return {
      result: runs.map((el) => ({
        ...el.toJSON(),
        car: cars.find(({ _id }) => _id.toString() === el.car.toString()).toJSON(),
        driver: drivers.find(({ _id }) => _id.toString() === el.driver.toString()).toJSON(),
      })),
    }
  },

  'run.count': async () => {
    const result = await runRepository.count()
    return { result }
  },
}

Код репозиторіїв:
const CarModel = require('../models/car')

class CarRepository {
  find(offset, limit, project = {}) {
    return CarModel.find({}, project, { skip: offset, limit, sort: { created_at: -1 } })
  }

  count() {
    return CarModel.countDocuments()
  }

  getByRunIds(ids) {
    return CarModel.find({ _id: { $in: ids } })
  }
}

module.exports = CarRepository

const DriverModel = require('../models/driver')

class DriverRepository {
  find(offset, limit, project = {}) {
    return DriverModel.find({}, project, { skip: offset, limit, sort: { created_at: -1 } })
  }

  count() {
    return DriverModel.countDocuments()
  }

  getByRunIds(ids) {
    return DriverModel.find({ _id: { $in: ids } })
  }
}

module.exports = DriverRepository

const RunModel = require('../models/run')

class RunRepository {
  find(offset, limit, project = {}) {
    return RunModel.find({}, project, { skip: offset, limit, sort: { created_at: -1 } })
  }

  create(run) {
    return RunModel.create({ ...run })
  }

  count() {
    return RunModel.countDocuments()
  }
}

module.exports = RunRepository
