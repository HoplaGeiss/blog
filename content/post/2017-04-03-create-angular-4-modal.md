---
title: "Angular 4 - Create a modal"
description: "Create a simple modal using angular 4"
tags:
  - angular4
date: "2017-04-03"
featured: false
slug: angular4-create-modal
image: /images/modal.png
---

After my previous post about creating a simple accordion with angular 4, I will now walk you through my implementation of a modal in angular 4.
To create the modal I used two files `modal.component.ts` and `modal.service.ts` and organised it in the usual component based architecture.

![Component architecture](/images/modal-architecture.png)

Demo: [http://www.muller.tech/hopla-modal/](http://www.muller.tech/hopla-modal/)

Source Code: [https://github.com/HoplaGeiss/hopla-modal](https://github.com/HoplaGeiss/hopla-modal)

Now let's take a look at both files.

``` typescript
// modal/modal.component.ts
import { Component, Input, OnInit, HostListener } from '@angular/core';
import { ModalService } from './modal.service';

@Component({
    selector: 'app-modal',
    styleUrls: ['./modal.scss'],
    template: `
      <div [ngClass]="{'closed': !isOpen}">
        <div class="modal-overlay" (click)="close(true)"></div>

        <div class="modal">
          <div class="title" *ngIf="modalTitle">
            <span class="title-text">{{ modalTitle }}</span>
            <span class="right-align" (click)="close(true)"><i class="material-icons md-24">clear</i></span>
          </div>

          <div class="body">
            <ng-content></ng-content>
          </div>
        </div>
      </div>
    `
})
export class ModalComponent implements OnInit {
  @Input() modalId: string;
  @Input() modalTitle: string;
  @Input() blocking = false;
  isOpen = false;

  @HostListener('keyup') onMouseEnter(event) {
    this.keyup(event);
  }

  constructor(private modalService: ModalService) {
  }

  ngOnInit() {
    this.modalService.registerModal(this);
  }

  close(checkBlocking = false): void {
    this.modalService.close(this.modalId, checkBlocking);
  }

  private keyup(event: KeyboardEvent): void {
    if (event.keyCode === 27) {
      this.modalService.close(this.modalId, true);
    }
  }
}
```

```typescript
// modal/modal.service.ts
import { Injectable } from '@angular/core';
import { ModalComponent } from './modal.component';

@Injectable()
export class ModalService {
  private modals: Array<ModalComponent>;

  constructor() {
    this.modals = [];
  }

  registerModal(newModal: ModalComponent): void {
    const modal = this.findModal(newModal.modalId);

    // Delete existing to replace the modal
    if (modal) {
      this.modals.splice(this.modals.indexOf(modal));
    }

    this.modals.push(newModal);
  }

  open(modalId: string): void {
    const modal = this.findModal(modalId);

    if (modal) {
      modal.isOpen = true;
    }
  }

  close(modalId: string, checkBlocking = false): void {
    const modal = this.findModal(modalId);

    if (modal) {
      if (checkBlocking && modal.blocking) {
        return;
      }

      modal.isOpen = false;
    }
  }

  private findModal(modalId: string): ModalComponent {
    for (const modal of this.modals) {
      if (modal.modalId === modalId) {
        return modal;
      }
    }
    return null;
  }
}
```

As you can see the user can pass the `modalTitle` and `blocking` as parameters to the modal. A blocking modal is a modal that can't be closed unless its the user completed an action, a non-blocking modal can be closed by clicking outside of the modal or by clicking the cross on top right.

Now lets see how to use this modal.

```typescript
// app.component.ts
import { Component } from '@angular/core';
import { ModalService } from './modal/modal.service';

@Component({
  selector: 'app-root',
  styleUrls: ['./app.component.scss'],
  template: `
    <button (click)='modalService.open(modalId)'>Open Modal</button>
    <app-modal [modalTitle]="'Some title'" [blocking]='false' [modalId]='modalId'>
      <div>Text inside the modal</div>
    </app-modal>
  `
})
export class AppComponent {
  modalId = 'hoplaModal';

  constructor(
    public modalService: ModalService
  ) {}
}

```

To finish here is some styling for our modal. Note the use of animation to display and hide the modal and the overlay!

``` scss
// modal/modal.component.scss
.modal-overlay {
  background-color: rgba(0, 0, 0, .4);
  bottom: 0;
  left: 0;
  position: fixed;
  right: 0;
  top: 0;
  z-index: 1000;
}

.closed {
  .modal{
    top: -50%;
  }
  .modal-overlay{
    display: none;
  }
}

.modal {
  box-shadow: 0 12px 15px 0 rgba(0, 0, 0, .22), 0 17px 20px 0 rgba(0, 0, 0, .12);

  background-color: white;
  left: calc(50% - 300px);
  max-height: calc(100% - 10em);
  overflow-y: auto;
  position: fixed;
  top: 5em;
  width: 600px;
  z-index: 1100;
  transition: all .5s ease;

  .title {
    background-color: red;
    text-align: center;
    color: white;
    line-height: 40px;

    .right-align {
      position: absolute;
      right: 5px;
      &, i{
        line-height: 40px;
      }

      &:hover{
        cursor: pointer;
        transform: scale(1.1);
      }
    }
  }

  .body{
    padding: 1.2em;
  }
}
```
