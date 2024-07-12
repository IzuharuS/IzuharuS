import { AfterViewInit, Component, OnInit } from '@angular/core';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.scss']
})
export class AppComponent implements OnInit, AfterViewInit {
  currentIndex = 0;
  startX = 0;
  currentX = 0;
  isDragging = false;
  cards: number [] = [];
  
  ngOnInit() {
    this.cards = [];
  }

  ngAfterViewInit() {
    this.cards = [];
    this.cards.push(1);
    this.cards.push(2);
    this.cards.push(3);
  }

  get transformStyle() {
    const offset = (-this.currentIndex * 100) / this.cards.length;
    return `translateX(${offset}%)`;
  }

  onTouchStart(event: TouchEvent) {
    this.startX = event.touches[0].clientX;
    this.isDragging = true;
  }

  onTouchMove(event: TouchEvent) {
    if (!this.isDragging) return;
    this.currentX = event.touches[0].clientX;
  }

  onTouchEnd() {
    if (!this.isDragging) return;
    const diffX = this.startX - this.currentX;
    if (diffX > 50 && this.currentIndex < this.cards.length - 1) {
      this.currentIndex++;
    } else if (diffX < -50 && this.currentIndex > 0) {
      this.currentIndex--;
    }
    this.isDragging = false;
  }
}

