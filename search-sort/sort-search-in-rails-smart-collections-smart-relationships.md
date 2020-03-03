# sort/search in rails smart collections / smart relationships

In this example I have a smart collection named clients. The records are fetched from a customers table in my database. The search and sort logic have been implemented for the index view of the clients collection \(see below\).

Each customer has many items from the orders table.

I declared an hasMany smart relationship between clients and orders. The search and sort logic have been implemented for the orders displayed as related data of a client record.

Here is the code declaring the collection and the smart relationship in the `lib/forest_liana/collections` folder in a `client.rb` file. Don't forget to use the is\_searchable and is\_sortable option to enable the input form for the search and the click on a collection header in the UI.

```text
module Forest
    class Client
      include ForestLiana::Collection

      collection :Client, is_searchable: true, is_sortable: true

      field :email, type: 'String'

      has_many :orders, reference: 'Order.id', is_sortable: true
    end
end
```

Here is the code implementing the get logic for clients and related orders \(including the search and sort\) in the `lib/forest_liana/controllers` folder in a `clients_controller.rb` file.

```text
module Forest
    class ClientController < ForestLiana::ApplicationController
      require 'jsonapi-serializers'

      before_action :set_params, only: [:index, :orders]

      class ClientSerializer
        include JSONAPI::Serializer

        def type
          'client'
        end

        def format_name(attribute_name)
          attribute_name.to_s.underscore
        end

        def unformat_name(attribute_name)
          attribute_name.to_s.dasherize
        end

        def self_link
            "/forest#{super}"
        end

        def relationship_related_link(attribute_name)
            "#{self_link}/relationships/#{format_name(attribute_name)}"
        end

        attributes :email
        has_many :orders
      end



      def index
        @clients = clients_query
        clients_count = @clients.count
        clients = @clients.limit(@limit).offset(@offset)
        clients_json = ClientSerializer.serialize(clients, is_collection: true, meta: {count: clients_count})
        render json: clients_json
      end

      def show
        client = Customer.find(params[:id])
        render json: ClientSerializer.serialize(client)
      end

      def orders
        @orders = orders_query
        orders = @orders.limit(@limit).offset(@offset)
        orders_count = @orders.count
        render json: serialize_models(orders, meta: {count: orders_count})
      end

      private

      def orders_query
        if @sort.present? && @search.present?
          Order.where(customer_id: params[:client_id]).where('id LIKE ?', "%#{@search}%").order(sort_query)
        elsif @sort.present? && !@search.present?
          Order.where(customer_id: params[:client_id]).order(sort_query)
        elsif !@sort.present? && @search.present?
          Order.where(customer_id: params[:client_id]).where('id LIKE ?', "%#{@search}%")
        else
          Order.where(customer_id: params[:client_id])
        end
      end

      def clients_query
        if @sort.present? && @search.present?
          Customer.where('email LIKE ?', "%#{@search}%").order(sort_query)
        elsif @sort.present? && !@search.present?
          Customer.all.order(sort_query)
        elsif !@sort.present? && @search.present?
          Customer.where('email LIKE ?', "%#{@search}%")
        else
          Customer.all
        end
      end

      def sort_query
        @sort = params[:sort]
        if @sort[0] === "-"
          @sort_trim = @sort[1..-1]
          return "#{@sort_trim} DESC"
        else
          return "#{@sort} ASC"
        end
      end      

      def set_params
        @limit = params[:page][:size].to_i
        @offset = (params[:page][:number].to_i - 1) * @limit
        @search = params[:search].to_s
        @sort = params[:sort]
      end
    end
  end
```

