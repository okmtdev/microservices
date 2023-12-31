# microservices

### Golang

```
# インストールできるバージョン
goenv install -l

# インストールする
goenv install 1.19.4

# バージョンを指定する
goenv global 1.19.4

# バージョン確認
goenv versions
  system
  1.19.0
* 1.19.4 (set by /Users/okmt/.goenv/version)
```

### Protocol Buffers

```
brew install protobuf
protoc --version
```

### Protocol Buffers Golang Plugins

```
go install google.golang.org/protobuf/cmd/protoc-gen-go@v1.28
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@v1.2

echo 'export PATH="$PATH:'$(go env GOPATH)'/bin"' >> ~/.zshrc
```

### Node.js

```
node --version
v21.2.0
```

### kubectl

```
kubectl version --client
Kustomize Version: v4.5.7
```

### kind

```
brew install kind
kind version
```

### istioctl

```
mkdir ~/development && cd ~/development
curl -L https://istio.io/downloadIstio | sh -
cd istio-1.20.0

tail -4 ~/.bashrc
##         ##
# istioctl  #
##         ##
export PATH="/Users/okmt/development/istio-1.20.0/bin:$PATH"

source ~/.bashrc

istioctl version
```

### generate data access interface

```
cd proto/book
protoc --go_out=. --go_opt=paths=source_relative \
  --go-grpc_out=. --go-grpc_opt=paths=source_relative \
  catalogue.proto
```

### implement backend server

```
go mod init gihyo/catelogue
go get google.golang.org/grpc
```

main.go の実装

```
// CatalogueServerのサービス実装用インターフェイスの構造体
type server struct {
	pb.UnimplementedCatalogueServer
}

// 自動生成された`catalogue_grpc.pb.go`の`GetBook`インターフェースを実装。
func (s *server) GetBook(ctx context.Context, in *pb.GetBookRequest) (*pb.GetBookResponse, error) {
	// リクエストで指定されたIDに応じて返す書籍情報を取得
	book := getBook(in.Id)

	// レスポンス用のデータを作成
	protoBook := &pb.Book{
		Id:     int32(book.Id),
		Title:  book.Title,
		Author: book.Author,
		Price:  int32(book.Price),
	}

	// レスポンス用のコードを使ってレスポンスを作り返却
	return &pb.GetBookResponse{Book: protoBook}, nil
}
```

gRPCCurl クライアントを利用して動作確認

```
brew install grpcurl
```

リフレクションサービスの登録

```
import (
    (省略)
    "google.golang.org/grpc/reflection"
)

func main() {
    (省略)

    relfection.Registor(s)
    (省略)
}
```

```
go run main.go
2023/12/11 01:25:46 server listening at [::]:50051
```

```
grpcurl -plaintext localhost:50051 list
book.Catalogue
grpc.reflection.v1alpha.ServerReflection

grpcurl -plaintext localhost:50051 list book.Catalogue
book.Catalogue.GetBook

grpcurl -plaintext -d '{"id": 1}' localhost:50051 book.Catalogue.GetBook
{
  "book": {
    "id": 1,
    "title": "The Awakening",
    "author": "Kate Chopin",
    "price": 1000
  }
}
```

### GraphQL

Apollo Server

```
npm install @apollo/server
```

GraphQL Schema を schema.js を定義する

```
export const typeDefs = `#graphql

# "Book"型の定義
type Book {
  id: Int
  title: String
  author: String
  price: Int
}

# クエリの定義
type Query {
  book(id: Int): Book
  books: [Book]
}
`;

```

resolver.js は現状モック

```
export const resolvers = {
  Query: {
    book: async (parent, args, context) => {
      const response = await context.dataSources.catalogueApi.getBook(args.id);
      return response;
    },
    books: async (parent, args, context) => {
      const response = await context.dataSources.catalogueApi.listBooks();
      return response;
    },
  },
};

```
