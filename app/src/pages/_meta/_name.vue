<script>
 // don't create a template for the section
import ListBook from '~/pages/ListBook.vue'
ListBook.head = function() {
    var name = decodeURIComponent(this.$route.params.name);
    var titles = {
        tag: `"རྟགས་བྱང་${name}”ཅན།`,
        series: `དཔེ་རྒྱུན་${name}`,
        rating: `སྐར་མ་${name}ཅན།`,
        author: `མཛད་པ་པོ་ནི་${name}`,
        publisher: `པར་ཁང་ནི་${name}`,
    }
    var meta = this.$route.path.split("/")[1];
    if ( titles[meta] !== undefined ) {
        return {
            title: titles[meta]
        }
    }
    return {}
}
export default ListBook
</script>
