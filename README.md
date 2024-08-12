# day-31-task
import React, { createContext, useState, useEffect } from 'react';
import { getBooks, getAuthors } from '../api';

const LibraryContext = createContext();

export const LibraryProvider = ({ children }) => {
  const [books, setBooks] = useState([]);
  const [authors, setAuthors] = useState([]);

  useEffect(() => {
    fetchBooks();
    fetchAuthors();
  }, []);

  const fetchBooks = async () => {
    const response = await getBooks();
    setBooks(response.data);
  };

  const fetchAuthors = async () => {
    const response = await getAuthors();
    setAuthors(response.data);
  };

  return (
    <LibraryContext.Provider value={{ books, authors, setBooks, setAuthors }}>
      {children}
    </LibraryContext.Provider>
  );
};

export default LibraryContext;
import React, { useContext } from 'react';
import LibraryContext from '../context/LibraryContext';

const BookList = ({ onEdit }) => {
  const { books, setBooks } = useContext(LibraryContext);

  const handleDelete = (id) => {
    setBooks(books.filter(book => book.id !== id));
  };

  return (
    <div>
      <h2>Books</h2>
      <table>
        <thead>
          <tr>
            <th>Title</th>
            <th>Author</th>
            <th>Actions</th>
          </tr>
        </thead>
        <tbody>
          {books.map(book => (
            <tr key={book.id}>
              <td>{book.title}</td>
              <td>{book.author}</td>
              <td>
                <button onClick={() => onEdit(book)}>Edit</button>
                <button onClick={() => handleDelete(book.id)}>Delete</button>
              </td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
};

export default BookList;
import React, { useContext } from 'react';
import LibraryContext from '../context/LibraryContext';

const AuthorList = ({ onEdit }) => {
  const { authors, setAuthors } = useContext(LibraryContext);

  const handleDelete = (id) => {
    setAuthors(authors.filter(author => author.id !== id));
  };

  return (
    <div>
      <h2>Authors</h2>
      <table>
        <thead>
          <tr>
            <th>Name</th>
            <th>Actions</th>
          </tr>
        </thead>
        <tbody>
          {authors.map(author => (
            <tr key={author.id}>
              <td>{author.name}</td>
              <td>
                <button onClick={() => onEdit(author)}>Edit</button>
                <button onClick={() => handleDelete(author.id)}>Delete</button>
              </td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
};

export default AuthorList;
import React, { useEffect, useContext } from 'react';
import { Formik, Form, Field, ErrorMessage } from 'formik';
import * as Yup from 'yup';
import LibraryContext from '../context/LibraryContext';

const BookForm = ({ bookToEdit, onClear }) => {
  const { books, setBooks } = useContext(LibraryContext);

  const initialValues = {
    title: '',
    author: '',
  };

  const validationSchema = Yup.object({
    title: Yup.string().required('Title is required'),
    author: Yup.string().required('Author is required'),
  });

  const handleSubmit = (values, { resetForm }) => {
    if (bookToEdit) {
      setBooks(
        books.map((book) =>
          book.id === bookToEdit.id ? { ...book, ...values } : book
        )
      );
    } else {
      setBooks([...books, { ...values, id: books.length + 1 }]);
    }
    resetForm();
    onClear();
  };

  useEffect(() => {
    if (bookToEdit) {
      initialValues.title = bookToEdit.title;
      initialValues.author = bookToEdit.author;
    }
  }, [bookToEdit]);

  return (
    <div>
      <h2>{bookToEdit ? 'Edit Book' : 'Add Book'}</h2>
      <Formik
        initialValues={initialValues}
        validationSchema={validationSchema}
        onSubmit={handleSubmit}
        enableReinitialize
      >
        <Form>
          <div>
            <label>Title:</label>
            <Field type="text" name="title" />
            <ErrorMessage name="title" component="div" />
          </div>
          <div>
            <label>Author:</label>
            <Field type="text" name="author" />
            <ErrorMessage name="author" component="div" />
          </div>
          <button type="submit">{bookToEdit ? 'Update' : 'Add'}</button>
          {bookToEdit && <button onClick={onClear}>Cancel</button>}
        </Form>
      </Formik>
    </div>
  );
};

export default BookForm;
import React, { useEffect, useContext } from 'react';
import { Formik, Form, Field, ErrorMessage } from 'formik';
import * as Yup from 'yup';
import LibraryContext from '../context/LibraryContext';

const AuthorForm = ({ authorToEdit, onClear }) => {
  const { authors, setAuthors } = useContext(LibraryContext);

  const initialValues = {
    name: '',
  };

  const validationSchema = Yup.object({
    name: Yup.string().required('Name is required'),
  });

  const handleSubmit = (values, { resetForm }) => {
    if (authorToEdit) {
      setAuthors(
        authors.map((author) =>
          author.id === authorToEdit.id ? { ...author, ...values } : author
        )
      );
    } else {
      setAuthors([...authors, { ...values, id: authors.length + 1 }]);
    }
    resetForm();
    onClear();
  };

  useEffect(() => {
    if (authorToEdit) {
      initialValues.name = authorToEdit.name;
    }
  }, [authorToEdit]);

  return (
    <div>
      <h2>{authorToEdit ? 'Edit Author' : 'Add Author'}</h2>
      <Formik
        initialValues={initialValues}
        validationSchema={validationSchema}
        onSubmit={handleSubmit}
        enableReinitialize
      >
        <Form>
          <div>
            <label>Name:</label>
            <Field type="text" name="name" />
            <ErrorMessage name="name" component="div" />
          </div>
          <button type="submit">{authorToEdit ? 'Update' : 'Add'}</button>
          {authorToEdit && <button onClick={onClear}>Cancel</button>}
        </Form>
      </Formik>
    </div>
  );
};

export default AuthorForm;
import React, { useState } from 'react';
import BookList from './BookList';
import BookForm from './BookForm';
import AuthorList from './AuthorList';
import AuthorForm from './AuthorForm';

const Main = () => {
  const [bookToEdit, setBookToEdit] = useState(null);
  const [authorToEdit, setAuthorToEdit] = useState(null);

  const clearEdit = () => {
    setBookToEdit(null);
    setAuthorToEdit(null);
  };

  return (
    <div>
      <div>
        <BookForm bookToEdit={bookToEdit} onClear={clearEdit} />
        <BookList onEdit={setBookToEdit} />
      </div>
      <div>
        <AuthorForm authorToEdit={authorToEdit} onClear={clearEdit} />
        <AuthorList onEdit={setAuthorToEdit} />
      </div>
    </div>
  );
};

export default Main;
body {
  font-family: Arial, sans-serif;
  padding: 20px;
}

table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 20px;
}

th, td {
  border: 1px solid #ddd;
  padding: 8px;
}

th {
  background-color: #f2f2f2;
}

button {
  padding: 5px 10px;
  margin-right: 5px;
  cursor: pointer;
}

form {
  margin-bottom: 20px;
}

input {
  padding: 5px;
  margin-right: 10px;
}
import React from 'react';
import { LibraryProvider } from './context/LibraryContext';
import Main from './components/Main';
import './styles.css';

function App() {
  return (
    <LibraryProvider>
      <Main />
    </LibraryProvider>
  );
}

export default App;
