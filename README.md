# LibManSys2
# Created Views for CRUD
#Created views in library/views.py:
from django.shortcuts import render, redirect
from .models import Book, Issue
from .forms import BookForm, IssueForm  
from django.shortcuts import get_object_or_404
from django.utils import timezone

def home(request):
    return render(request, 'library/home.html')

def add_book(request):
    if request.method == "POST":
        form = BookForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect('view_books')
    else:
        form = BookForm()
    return render(request, 'library/add_book.html', {'form': form})

def view_books(request):
    books = Book.objects.all()
    return render(request, 'library/view_books.html', {'books': books})



def update_book(request, book_id):
    book = get_object_or_404(Book, id=book_id)
    form = BookForm(request.POST or None, instance=book)
    if form.is_valid():
        form.save()
        return redirect('view_books')
    return render(request, 'library/update_book.html', {'form': form})

def delete_book(request, book_id):
    book = get_object_or_404(Book, id=book_id)
    if request.method == "POST":
        book.delete()
        return redirect('view_books')
    return render(request, 'library/delete_book.html', {'book': book})


def issue_book(request):
    if request.method == "POST":
        form = IssueForm(request.POST)
        if form.is_valid():
            form.save()
            # Mark book as not available
            form.cleaned_data['book'].available = False
            form.cleaned_data['book'].save()
            return redirect('issued_books')
    else:
        form = IssueForm()
    return render(request, 'library/issue_book.html', {'form': form})

def issued_books(request):
    issues = Issue.objects.all()
    return render(request, 'library/issued_books.html', {'issues': issues})

def return_book(request, issue_id):
    issue = get_object_or_404(Issue, id=issue_id)
    if request.method == "POST":
        issue.return_date = timezone.now()
        issue.book.available = True
        issue.book.save()
        issue.save()
        return redirect('issued_books')
    return render(request, 'library/return_book.html', {'issue': issue})



# Create Forms
#In library/forms.py:
from django import forms
from .models import Book, Issue

class BookForm(forms.ModelForm):
    class Meta:
        model = Book
        fields = ['title', 'author', 'isbn', 'available']

class IssueForm(forms.ModelForm):
    class Meta:
        model = Issue
        fields = ['book', 'student']

# Created Templates
#Created these HTML files inside library/templates/library/:

#home.html:
#add_book.html
#view_books.html
#view_books.html
#update_book.html
#delete_book.html
#issue_book.html
#issued_books.html
#return_book.html



 # Created urls.py inside library app
# inside library/urls.py:
from django.urls import path
from . import views

urlpatterns = [
    path('', views.home, name='home'),
    path('add/', views.add_book, name='add_book'),
    path('books/', views.view_books, name='view_books'),
    path('update/<int:book_id>/', views.update_book, name='update_book'),
    path('delete/<int:book_id>/', views.delete_book, name='delete_book'),
    path('issue/', views.issue_book, name='issue_book'),
    path('issued/', views.issued_books, name='issued_books'),
    path('return/<int:issue_id>/', views.return_book, name='return_book'),

# Connected app's URLs to the main project
#library_system/urls.py (main project file):
from django.contrib import admin
from django.urls import path, include   # include is important!

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('library.urls')),  # This connects app URLs!
]


