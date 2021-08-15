## React ApllloClient and GraphQL sample

- please install

```bash
yarn add @apollo/react-hooks
yarn add graphql
```

### setup AplloClient

- App.js

Apollo Client is a comprehensive state management library for JavaScript that enables you to manage both local and remote data with GraphQL.

```js
import { ApolloProvider } from "@apollo/react-hooks";
import { ApolloClient, InMemoryCache } from "@apollo/client";

const client = new ApolloClient({
  uri: "http://127.0.0.1:8000/graphql/",
  headers: {
    authorization: localStorage.getItem("token")
      ? `JWT ${localStorage.getItem("token")}`
      : "",
  },
  cache: new InMemoryCache(),
});
```

### creating query example

- mutation

```js
import gql from "graphql-tag";

export const CREATE_USER = gql`
  mutation ($username: String!, $password: String!) {
    createUser(input: { username: $username, password: $password, email: "" }) {
      user {
        id
        username
      }
    }
  }
`;

//component

import { GET_TOKEN, CREATE_USER } from "../queries";
import { useMutation } from "@apollo/client";

const [getToken] = useMutation(GET_TOKEN);
const [createUser] = useMutation(CREATE_USER);

//ex
const authUser = async (e) => {
  e.preventDefault();
  if (isLogin) {
    try {
      const result = await getToken({
        variables: { username: username, password: password },
      });
      localStorage.setItem("token", result.data.tokenAuth.token);
      result.data.tokenAuth.token && (window.location.href = "/employees");
    } catch (err) {
      alert(err.message);
    }
  } else {
  }
};
//check jwt token
useEffect(() => {
  if (localStorage.getItem("token")) {
    const decodedToken = jwtDecode(localStorage.getItem("token"));
    if (decodedToken.exp * 1000 < Date.now()) {
      localStorage.removeItem("token");
    } else {
      window.location.href = "/employees";
    }
  }
}, []);

return (
  <>
    <form onSubmit={authUser}></form>
  </>
);
```

- query

```js
export const GET_EMPLOYEES = gql`
  query {
    allEmployees {
      edges {
        node {
          id
          name
          joinYear
          department {
            id
            deptName
          }
        }
      }
    }
  }
`;

//component
import { useQuery } from "@apollo/react-hooks";
const {
  loading: loadingEmployees,
  data: dataEmployees,
  error: errorEmployees,
} = useQuery(GET_EMPLOYEES);
```

- singlequery

useLazyQuery makes it very easy to execute a GraphQL query at any given time.

```js
export const GET_SINGLE_EMPLOYEE = gql`
  query ($id: ID!) {
    employee(id: $id) {
      id
      name
      joinYear
      department {
        id
        deptName
      }
    }
  }
`;

import { useLazyQuery } from "@apollo/client";

const [
  getSingleEmployee,
  { data: dataSingleEmployee, error: errorSingleEmployee },
] = useLazyQuery(GET_SINGLE_EMPLOYEE, {
  fetchPolicy: "network-only",
});
```
