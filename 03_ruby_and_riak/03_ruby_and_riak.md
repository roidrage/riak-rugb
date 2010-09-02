!SLIDE

# Riak and Ruby #

!SLIDE bullets incremental

# Scenarios #

* Session store
* Cache
* File storage
* Store data with well-known keys

!SLIDE

# Ripple #

!SLIDE code small ruby

# Riak API #

    @@@ ruby
    require 'riak/map_reduce'

    client = Riak::Client.new
    Riak::MapReduce.new(client).
      add("rubies", "rbx").
      add("rubies", "mri").
      map("Riak.mapValues", :keep => true).run

!SLIDE code small ruby

# Mapping Ruby Objects #

    @@@ ruby
    require 'ripple' 

    class Ruby
      include Ripple::Document
      property :version, String
    end

    Ruby.find("rbx") # => <Ruby:rbx version="1.0.0">
