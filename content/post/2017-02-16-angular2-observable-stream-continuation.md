---
title: "Angular2 observable - Stream continuation"
description: "Guide on how to keep an observable's stream up and running when an error occurs."
tags:
  - angular2
  - observable
date: "2017-02-16"
featured: true
slug: angular2-observable-stream-continuation
image: /images/stream.jpeg
---

Keeping an observable stream up and running is a common use case for many application. Lets take the example of an application with an input field taking a location as parameter which then calls foursquare's API at each key stroke to query the recommended location nearby. If an HTTP call to foursquare fails, we want our observable stream to stay up and running.

```typescript
// home.component.ts
@Component({
  selector: 'vs-home',
  template: `
    <div class="home">
      <form [formGroup]="form">
        <md-input-container class="form-group" >
          <input md-input class="form-control" formControlName="location" placeholder="Choose a Location">
        </md-input-container>
      </form>

      <h2>Recommended venues close to this location:</h2>

      <div *ngFor="let venue of venues | async" >
        <vs-venue [venue]="venue"></vs-venue>
      </div>
    </div>
  `
})
export class HomeComponent implements OnInit {
  venues: Observable<Venue[]>;
  form: any;

  constructor(
    private venueService: VenueService,
    private formBuilder: FormBuilder
  ) {}

  ngOnInit() {
    this.form = this.formBuilder.group({
      location: '',
    });

    this.venues = this.form.valueChanges
      .debounceTime(400)
      .distinctUntilChanged()
      .switchMap(term => this.venueService.search(term.location))
      .catch(this.handleError);
  }
}
```

```typescript
// venue.service.component.ts
search(term: string): Observable<Venue[]> {
  const url = 'https://api.foursquare.com/v2/venues/explore?section=topPicks';
  return this.http
    .get(`${url}&near=${term}`)
    .map(this.extractData)
    .catch(this.handleError);
}

private extractData(res: Response) {
    const body = res.json().response;
    return body.map(data => new Venue(data.venue.id, data.venue.name));
}

private handleError (error: Response | any) {
   return Observable.throw(errMsg);
}
```
The typical way to deal with errors when doing an http call is to catch errors. Since we are using observables, the `handleError` method needs to either return an Observable or throw an error.

Throwing an error, stops the observable's stream. Concretely, this means that after the user enters a value returning an error from foursquare, the next value he enters in the input wouldn't call the API anymore.

### Solutions

To fix this, we need to find a way not to stop the stream.

#### Solution 1
The first solution is to remove the catch and call `onErrorResumeNext`.

```typescript
// venue.service.component.ts
search(term: string): Observable<Venue[]> {
  const url = 'https://api.foursquare.com/v2/venues/explore?section=topPicks';
  return this.http
    .get(`${url}&near=${term}`)
    .map(this.extractData)
    .onErrorResumeNext
}

private extractData(res: Response) {
    const body = res.json().response;
    return body.map(data => new Venue(data.venue.id, data.venue.name));
}
```

However this solution has its limits, we continue the stream, but we don't change the venues displayed, which is a little weird. The user would input a location, get the venues nearby and then if he changes the input and in case an error occurs he wouldn't be aware of it as the venues would stay the same.

#### Solution 2
The second solution is to keep the catch, but return a different Observable without venues.

```typescript
// venue.service.component.ts
search(term: string): Observable<Venue[]> {
  const url = 'https://api.foursquare.com/v2/venues/explore?section=topPicks';
  return this.http
    .get(`${url}&near=${term}`)
    .map(this.extractData)
    .onErrorResumeNext
}

private extractData(res: Response) {
    const body = res.json().response;
    return body.map(data => new Venue(data.venue.id, data.venue.name));
}

private handleError (error: Response | any) {
  return Observable.of([]);
}
```
